<script lang="ts" setup>
import PInstanceList from '@/components/Admin/Instance/PInstanceList.vue';
import {computed, nextTick, ref} from 'vue';
import {admin} from '@/class/Client';
import {useRoute, useRouter} from 'vue-router';
import {useAuthData} from '@/stores/AuthStore';
import PInstanceEditor from '@/components/Admin/Instance/PInstanceEditor.vue';
import PageContainer from '@/components/PageContainer.vue';
import PageHeader from '@/components/PageHeader.vue';

const data = ref([]);
const load = () => admin.ins.list().then(res => {
    data.value = res.data.data;
});
const route = useRoute();
const router = useRouter();

let id: any;

useAuthData().listen(() => {
    load()
    nextTick(() => id = computed({
        get: () => route.params.insId,
        set: (v) => router.isReady().then(() => router.push({name: 'admin.instance.index', params: {insId: v}}))
    }));
});
</script>

<template>
    <PageContainer>
        <PageHeader :breadcrumb="['管理面板', '实例管理']">
            <template #title>实例管理</template>
            <template #action>
                <n-button type="primary" text-color="white" @click="id = 'create'">新建实例</n-button>
            </template>
        </PageHeader>
    </PageContainer>

    <n-divider class="my-0"/>

    <div class="mdui-container mt-4">
        <PInstanceList :data="data"/>
        <PInstanceEditor v-model:id="id" @reload="load"/>
    </div>
</template>