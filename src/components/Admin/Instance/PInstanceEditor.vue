<script lang="ts" setup>
import {computed, ref, watch} from 'vue';
import {admin} from '@/class/Client';
import {DropdownOption, useMessage} from 'naive-ui';
import type {MentionOption} from 'naive-ui';
import AllowSubmit from '@/components/AllowSubmit.vue';
import PSwitch from '@/components/PSwitch.vue';
import PSelector from '@/components/PSelector.vue';

const props = defineProps(['id']);
const emits = defineEmits(['update:id', 'reload']);
const message = useMessage();

const initData = {
    id: 0,
    uuid: '',
    name: '',
    description: '',
    is_suspended: 0,
    relationship: {
        user_id: 0
    },
    node_id: '',
    node_allocation_id: '',
    app_id: '',
    app_version_id: '',
    cpu: 0,
    memory: 0,
    swap: 0,
    disk: 0,
    image: '',
    created_at: 0,
    updated_at: 0
};
const data = ref(initData);
const user = ref('@');
const users = ref<MentionOption[]>([]);
const nodes = ref<MentionOption[]>();
const apps = ref<MentionOption[]>();
const images = ref<DropdownOption[]>([]);
const allocations = ref([]);
const versions = ref([]);
const actions = {
    init() {
        data.value = initData;
        actions.load();
    },
    load() {
        admin.node.list().then(res => {
            nodes.value = res.data.data.map((v: { id: number, name: string }) => {
                return {label: v.id + '-' + v.name, value: v.id};
            });
        });
        admin.app.list().then(res => {
            apps.value = res.data.data.map((v: { id: number, name: string }) => {
                return {label: v.id + '-' + v.name, value: v.id};
            });
        });
        if (create.value) return;
        admin.ins.detail(props.id).then(res => {
            data.value = res.data.attributes;
            admin.user.detail(data.value.relationship.user_id).then(res => {
                user.value = '@' + res.data.attributes.name;
            });
        });
    },
    confirm() {
        if (create.value) {
            admin.ins.create(data.value).then(() => {
                actions.close('实例创建成功。', true);
            });
        } else {
            admin.ins.update(props.id, data.value).then(() => {
                actions.close('实例修改成功。', true);
            });
        }
    },
    delete() {
        admin.ins.delete(props.id).then(() => {
            actions.close('实例删除成功。', true);
        });
    },
    close(msg?: string, reload?: boolean) {
        if (msg) message.success(msg);
        if (reload) emits('reload');
        emits('update:id', null);
    },
    onAppSelect(option: MentionOption) {
        admin.app.detail(option.value! as unknown as number).then(res => {
            images.value = JSON.parse(res.data.attributes.images).map((v: string, id: number) => {
                if (!id) data.value.image = v;
                return {label: v, value: v}
            });
        });
        admin.app.listVersions(option.value! as unknown as number).then(res => {
            versions.value = res.data.data.map((v: { id: number, name: string }, id: number) => {
                if (!id) data.value.app_version_id = v.id.toString();
                return {label: v.id + '-' + v.name, value: v.id};
            });
        });
    },
    onImageSelect(key: string | number, option: DropdownOption) {
        data.value.image = option.value as string;
    },
    onNodeSelect(option: MentionOption) {
        admin.node.listAllocations(option.value! as unknown as number, {unused: 1}).then(res => {
            allocations.value = res.data.data.map((v: { id: number, alias: string, port: number }) => {
                return {label: v.id + '-' + v.alias + ':' + v.port, value: v.id};
            });
        });
    },
    onUserSelect(option: MentionOption) {
        data.value.relationship.user_id = option.uid as unknown as number;
        console.log(option.uid);
    }
};
const width: number = window.innerWidth;
const create = computed(() => props.id == 'create');

watch(() => props.id, (v) => !v || actions.init());
watch(() => user.value, (v: string) => {
    if (!v.length) user.value = '@';
    if (v.length > 3) {
        if (users.value!.length) return;
        admin.user.list({name: v.substring(1)}).then(res => {
            users.value = res.data.data.map((v: { id: number, name: string }) => {
                return {label: v.name, value: v.name, uid: v.id};
            });
        });
    } else {
        users.value = [];
    }
});
</script>

<template>
    <n-drawer :default-width="Math.min(768, width)" resizable placement="right" :mask-closable="false"
              :show="!!id" @update:show="actions.close">
        <n-drawer-content closable :native-scrollbar="false">
            <template #header>{{ create ? '创建实例' : '修改实例' }}</template>

            <n-form :model="data" label-placement="left" :label-width="84" class="mt-2"
                    @submit.prevent="actions.confirm()">
                <n-form-item label="ID" v-if="!create">
                    <n-input-number v-model:value="data.id" readonly/>
                </n-form-item>

                <n-form-item label="名称">
                    <n-input v-model:value="data.name" placeholder="输入实例名称"/>
                </n-form-item>

                <n-form-item label="UUID" v-if="!create">
                    <n-input v-model:value="data.uuid" readonly/>
                </n-form-item>

                <n-form-item label="实例简介">
                    <n-input v-model:value="data.description" placeholder="输入实例简介" type="textarea"/>
                </n-form-item>

                <n-form-item label="所属用户">
                    <n-mention v-model:value="user" :options="users" default-value="@"
                               :on-select="actions.onUserSelect">
                        <template #empty>
                            {{ user.length > 3 ? '无结果' : '输入至少3个字符以检索' }}
                        </template>
                    </n-mention>
                </n-form-item>

                <n-form-item label="节点">
                    <PSelector :list="nodes" v-model="data.node_id" :onSelect="actions.onNodeSelect"/>
                </n-form-item>

                <n-form-item label="节点端口" v-if="data.node_id">
                    <PSelector :list="allocations" v-model="data.node_allocation_id"/>
                </n-form-item>

                <n-form-item label="应用">
                    <PSelector :list="apps" v-model="data.app_id" :onSelect="actions.onAppSelect"/>
                </n-form-item>

                <n-form-item label="应用版本" v-if="!!data.app_id">
                    <PSelector :list="versions" v-model="data.app_version_id"/>
                </n-form-item>

                <n-form-item label="应用镜像" v-if="!!data.app_id">
                    <n-input-group>
                        <n-dropdown trigger="click" placement="bottom-start" :options="images"
                                    @select="actions.onImageSelect">
                            <n-button strong type="primary">选择镜像</n-button>
                        </n-dropdown>
                        <n-input v-model:value="data.image" placeholder="输入实例 Docker 镜像"/>
                    </n-input-group>
                </n-form-item>

                <n-form-item label="暂停状态" v-if="!create">
                    <PSwitch v-model="data.is_suspended"/>
                </n-form-item>

                <n-form-item label="CPU 限制">
                    <n-input-number v-model:value="data.cpu" placeholder="输入实例 CPU 限制"/>
                    <span class="ml-3">%</span>
                </n-form-item>

                <n-form-item label="内存限制">
                    <div>
                        <n-space class="items-center">
                            <n-input-number v-model:value="data.memory" min="0"
                                            placeholder="输入实例内存限制"/>
                            MB
                            <n-input-number v-model:value="data.swap" min="0"
                                            placeholder="输入实例虚拟内存限制"/>
                            MB
                        </n-space>
                    </div>
                </n-form-item>

                <n-form-item label="硬盘限制">
                    <n-input-number v-model:value="data.disk" placeholder="输入实例硬盘限制" min="0" :step="1024"/>
                    <span class="ml-3">MB</span>
                </n-form-item>

                <n-form-item label="更新时间" v-if="!create">
                    <n-time :time="new Date(data.updated_at)"/>
                </n-form-item>

                <n-form-item label="创建时间" class="-mt-4" v-if="!create">
                    <n-time :time="new Date(data.created_at)"/>
                </n-form-item>

                <AllowSubmit/>
            </n-form>

            <template #footer>
                <n-button type="error" ghost @click="actions.delete()" v-if="!create">删除</n-button>
                <n-button type="primary" text-color="white" @click="actions.confirm()">{{
                        create ? '创建' : '修改'
                    }}
                </n-button>
            </template>
        </n-drawer-content>
    </n-drawer>
</template>