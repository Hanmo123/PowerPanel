<script lang="ts" setup>
import {useInstanceDetail} from '@/stores/Instance/DetailStore';
import {useRoute, useRouter} from 'vue-router';
import {useAuthData} from '@/stores/AuthStore';
import {ins} from '@/class/Client';
import {computed, onUnmounted} from 'vue';
import PageContainer from '@/components/PageContainer.vue';
import PageHeader from '@/components/PageHeader.vue';
import {InstanceStatus as S} from '@/class/Constant/Status';
import {useWebSocketStore} from '@/stores/Instance/WebSocketStore';
import {InfoFilled} from '@vicons/material';

const detail = useInstanceDetail();
const WebSocketStore = useWebSocketStore();
const route = useRoute();
const insId = route.params.insId as unknown as number;
const routeName = computed(() => route.name as 'instance.detail' | 'instance.console' | 'instance.file');
const layoutName = computed(() => {
    return {
        'instance.detail': '实例信息',
        'instance.console': '控制台',
        'instance.file': '文件管理',
        'instance.action': '实例操作'
    }[routeName.value];
});
const router = useRouter();

useAuthData().listen(() => {
    ins.fetch(insId).then(res => {
        detail.init(res.data.attributes);
    });
});

onUnmounted(() => {
    detail.$reset();
});

const optionList = [
    {key: 'start', label: '启动', rule: [S.STOPPED]},
    {key: 'stop', label: '关闭', rule: [S.RUNNING]},
    {key: 'restart', label: '重启', rule: [S.RUNNING]},
    {key: 'kill', label: '终止', rule: [S.RUNNING, S.STARTING, S.STOPPING]}
];
const options = computed(() => {
    return optionList.filter((row) => {
        return row.rule.indexOf(detail.status!) !== -1;
    });
});
const onPowerAction = (key: 'start' | 'stop' | 'restart' | 'kill') => {
    WebSocketStore.send({
        type: 'power',
        data: key
    });
}

const tab = computed({
    get() {
        return routeName.value
    },
    async set(to: string) {
        await router.isReady();
        await router.push({name: to, params: {insId: insId}});
    }
})
</script>

<template>
    <PageContainer>
        <PageHeader :breadcrumb="['控制面板', detail.name || detail.uuid?.split('-')[0], layoutName]">
            <template #title>{{ layoutName }}</template>
            <template #action>
                <div class="action" v-if="routeName === 'instance.console'">
                    <n-dropdown trigger="click" :options="options" class="min-w-[96px]" placement="bottom-end"
                                @select="onPowerAction">
                        <n-button type="primary" text-color="white">操作</n-button>
                    </n-dropdown>
                </div>
            </template>
            <template #tab>
                <n-tabs :bar-width="28" type="line" class="-mb-[1px] mt-2" v-model:value="tab">
                    <n-tab name="instance.detail">
                        实例信息
                    </n-tab>
                    <n-tab name="instance.console">
                        控制台
                    </n-tab>
                    <n-tab name="instance.file">
                        文件管理
                    </n-tab>
                    <n-tab name="instance.action">
                        实例操作
                    </n-tab>
                </n-tabs>
            </template>
        </PageHeader>
    </PageContainer>

    <n-divider class="my-0"/>

    <div class="mdui-container mt-4" v-if="detail.is_suspended">
        <n-alert title="实例已被暂停" class="alert-info" type="info">
            <template #icon>
                <n-icon>
                    <InfoFilled/>
                </n-icon>
            </template>
            此实例已被设置为暂停，部分操作将不可用。
        </n-alert>
    </div>

    <RouterView/>
</template>