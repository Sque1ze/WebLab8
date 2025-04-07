<template>
    <UCard class="w-full" :ui="{
        base: '',
        ring: '',
        divide: 'divide-y divide-gray-200 dark:divide-gray-700 h-min-full h-full',
        header: { padding: 'px-4 py-5' },
        body: { padding: '', base: 'divide-y divide-gray-200 dark:divide-gray-700' },
        footer: { padding: 'p-4' }
    }">
        <div v-if="total > 0" class="flex justify-end px-3 py-3.5 border-t border-gray-200 dark:border-gray-700">
            <div class="me-auto">
                <button
                    class="py-3 px-4 mx-2 inline-flex items-center justify-between gap-x-2 text-sm font-semibold rounded-lg border border-transparent bg-teal-100 text-teal-800 hover:bg-teal-200 disabled:opacity-50 disabled:pointer-events-none dark:hover:bg-teal-900 dark:text-teal-500 dark:hover:text-teal-400"
                    @click="toggleSwitcher">Змінити таблицю</button>
                <NuxtLink :to="'/post'"
                    class="mx-2 py-3 px-4 inline-flex items-center gap-x-2 text-sm font-semibold rounded-lg border border-transparent bg-blue-100 text-blue-800 hover:bg-blue-200 disabled:opacity-50 disabled:pointer-events-none dark:hover:bg-blue-900 dark:text-blue-400">
                    Створити
                </NuxtLink>
            </div>
            <UPagination v-model="page" :page-count="pageCount" :total="total" />
        </div>
        <div v-if="posts">
            <NuxtTable v-if="switcher && posts" :posts="posts" :columns="columns" :page="page" @handleDelete="deletePost"></NuxtTable>
            <Table v-else="switcher" :posts="posts"></Table>
        </div>
        <Loader v-else />
    </UCard>
</template>

<script setup lang="ts">
import Loader from "~/components/loader/index.vue"
import Table from "./components/table/index.vue";
import NuxtTable from "./components/nuxtTable/index.vue";
import type { IFunctional, IPost } from "~/types";

const columns: IFunctional[] = [{
    key: 'id',
    label: '#',
}, {
    key: 'user.name',
    label: `Автор`,
}, {
    key: 'category.title',
    label: 'Категорія',
}, {
    key: 'title',
    label: 'Заголовок',
}, {
    key: 'published_at',
    label: 'Опубліковано',
}, {
    key: 'actions',
    label: '',
}];

const page = ref<number>(1);
const total = ref<number>(0);
const pageCount = ref<number>(9);
const posts = ref<[IPost]>();

const getPosts = async () => {
    await $fetch<{ data: [IPost], total: number }>(`http://localhost:8000/api/blog/posts`, {
        query: {
            'page': page.value,
            'per_page': pageCount.value,
        }
    }).then(response => {
        posts.value = response.data;
        total.value = response.total;
    });
}

watch(page, async () => {
    await getPosts();
});

await getPosts();

const deletePost = async (row: IPost) => {
    await $fetch(`http://localhost:8000/api/blog/post/${row.id}`, {
        method: 'delete'
    }).then(async (response) => {
        await getPosts();
    });
}

const switcher = ref<boolean>(true);
const toggleSwitcher = () => {
    switcher.value = !switcher.value;
};
</script>
