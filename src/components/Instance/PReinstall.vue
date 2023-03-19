<script lang="ts" setup>
import {useInstanceDetail} from '@/stores/Instance/DetailStore';
import {ins} from '@/class/Client';

const confirm = () => {
    ins.reinstall(useInstanceDetail().id!);
};
</script>

<template>
    <n-card title="重新安装实例">
        <div class="-mt-2">
            此操作将重新执行实例镜像的安装脚本。请注意：此操作可能导致数据丢失，建议重装前先行备份数据。
            <div class="text-right mt-2">
                <n-popconfirm @positive-click="confirm" positive-text="确认" negative-text="取消">
                    <template #trigger>
                        <n-button type="primary">重装实例</n-button>
                    </template>
                    确认重新安装实例操作？
                </n-popconfirm>
            </div>
        </div>
    </n-card>
</template>