<template>
  <div class="overflow-hidden rounded-[0.5rem] border bg-background shadow hidden space-y-6 p-10 pb-16 md:block m-12">
    <ListBookmarks
      v-model="searchValue"
      :params="{
				search: searchValue,
        page: nextPage,
        perPage: perPage,
      }"
      :items="data?.data?? []"
      @set-page-index="onSetPageIndex"
      @previous-page="onPreviousPage"
      @next-page="onNextPage"
      @per-page="onPerPage"
    />
    <AddBookmark />
  </div>
</template>

<script setup lang="ts">
import { useQuery } from '@tanstack/vue-query'
import { ref } from "vue";

import { AddBookmark, ListBookmarks } from "@/components/bookmarks";

// variables
const searchValue = ref('')
const nextPage = ref(1)
const perPage = ref(2)

const onSetPageIndex = (page: number) => {
	nextPage.value = page
}
const onPreviousPage = () => {
	nextPage.value--
}
const onNextPage = () => {
	nextPage.value++
}
const onPerPage = (value: number) => {
	perPage.value = value
}

const fetcher = async () =>{
	// TODO: create a interceptor to handle the base url
	return await fetch(`http://localhost:8000/bookmarks?search=${searchValue.value}&page=${nextPage.value}&perPage=${perPage.value}`)
		.then((response) => response.json(),
	)
}

const { data, suspense } = useQuery({
	queryKey: ['bookmarks', {
		search: searchValue,
		page: nextPage,
		perPage: perPage
	}],
	queryFn: fetcher,
	placeholderData: (previousData: any) => previousData,
	enabled: true,
})
// await suspense()
</script>
