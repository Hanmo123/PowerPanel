<script lang="ts" setup>
import {UploadFileInfo, useMessage} from 'naive-ui';
import {ref, watch} from 'vue';
import {admin} from '@/class/Client';
import PSelector from '@/components/PSelector.vue';

const props = defineProps(['show']);
const emits = defineEmits(['reload', 'update:show']);
const message = useMessage();
const gameId = ref('');
const content = ref();
const list = ref([]);

async function onChange(options: { fileList: UploadFileInfo[] }) {
    content.value = JSON.parse(await options.fileList[0].file?.text()!);
}

function onSubmit() {
    if (!content.value) message.warning('请先选择文件');
    admin.app.import(gameId.value as unknown as number, JSON.stringify(content.value)).then(() => {
        emits('reload');
        message.success('导入成功');
    });
}

watch(() => props.show, async (v: boolean) => {
    if (v && !list.value.length)
        list.value = (await admin.app.game.list()).data.data.map((v: { id: number, name: string }) => {
            return {label: v.id + '-' + v.name, value: v.id};
        });
});
</script>

<template>
    <n-modal preset="dialog" title="导入镜像" positive-text="导入" :show-icon="false" :auto-focus="false"
             @positive-click="onSubmit" :show="show" @update:show="(v: boolean) => emits('update:show', v)">
        <n-form label-placement="left" class="mt-6 -mb-4">
            <n-form-item label="所属游戏">
                <PSelector v-model="gameId" :list="list"/>
            </n-form-item>
            <n-form-item label="镜像文件">
                <n-upload abstract @change="onChange">
                    <n-upload-trigger #="{ handleClick }" abstract>
                        <n-input-group @click="handleClick">
                            <n-input readonly :value="content?.name" placeholder="请先上传镜像文件..."></n-input>
                            <n-button type="primary" secondary>
                                上传镜像
                            </n-button>
                        </n-input-group>
                    </n-upload-trigger>
                </n-upload>
            </n-form-item>
        </n-form>
    </n-modal>
</template>