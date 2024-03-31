<template>
  <div>
    <h2 class="text-2xl font-bold mb-4">
      Bookmarks
    </h2>

    <div class="bg-background/95 py-4 backdrop-blur supports-[backdrop-filter]:bg-background/60">
      <form>
        <div class="relative">
          <Search class="absolute left-2 top-2.5 size-4 text-muted-foreground" />
          <Input
            v-model="proxyValue"
            placeholder="Search"
            class="pl-8"
          />
        </div>
      </form>
    </div>

    <div class="flex-1 flex flex-col gap-2">
      <TransitionGroup
        name="list"
        appear
      >
        <button
          v-for="item of filteredMailList"
          :key="item.id"
          :class="cn(
            'flex flex-col items-start gap-2 rounded-lg border p-3 text-left text-sm transition-all hover:bg-accent',
            selectedMail === item.id && 'bg-muted',
          )"
          @click="selectedMail = item.id"
        >
          <div class="flex w-full flex-col gap-1">
            <div class="flex items-center">
              <div class="flex items-center gap-2">
                <div class="font-semibold">
                  {{ item.title }}
                </div>
                <span
                  v-if="!item.read"
                  class="flex h-2 w-2 rounded-full bg-blue-600"
                />
              </div>
              <div
                :class="cn(
                  'ml-auto text-xs',
                  selectedMail === item.id
                    ? 'text-foreground'
                    : 'text-muted-foreground',
                )"
              >
                {{ formatDistanceToNow(new Date(item.createdAt), { addSuffix: true }) }}
              </div>
            </div>

            <div class="text-xs font-medium">
              {{ item.url }}
            </div>
            <div class="text-xs font-medium">
              {{ extractTwitterUsername(item.url) }}
            </div>
          </div>
          <div class="line-clamp-2 text-xs text-muted-foreground">
            {{ item.description.substring(0, 300) }}
          </div>
          <div class="flex items-center gap-2">
            <Badge
              v-for="label of ['work', 'personal', 'social', 'important']"
              :key="label"
              :variant="getBadgeVariantFromLabel(label)"
            >
              {{ label }}
            </Badge>
          </div>
        </button>
      </TransitionGroup>
    </div>

    <DataTablePagination
      :table="{
        getFilteredRowModel: () => ({
          rows: filteredMailList?.value ?? [],
        }),
        getFilteredSelectedRowModel: () => ({
          rows: filteredMailList?.value?.filter((item) => item.selected) ?? [],
        }),
        ...table
      }"
    />
  </div>
</template>

<script lang="ts" setup>
import {
	Search,
} from 'lucide-vue-next'

import { computed, ref } from 'vue'
import { refDebounced } from '@vueuse/core'
import { formatDistanceToNow } from 'date-fns'


import { Input } from '@/components/ui/input'
import { Badge } from '@/components/ui/badge'
import { cn } from '@/lib/utils'

import DataTablePagination from "~/components/bookmarks/DataTablePagination.vue";

interface MailListProps {
	modelValue: number | string;
	items: any[]
	params: {
		search: string,
		page: number,
		perPage: number,
	}
}

const props = defineProps<MailListProps>()
const selectedMail = defineModel<string>('selectedMail', { required: false })

const emits = defineEmits<{
	(e: 'update:modelValue', payload: string | number): void
	(e: 'setPageIndex', payload: number): void
	(e: 'previousPage', payload: number): void
	(e: 'nextPage', payload: number): void
	(e: 'perPage', payload: number): void
}>()

const proxyValue = computed({
	get() {
		return props.modelValue;
	},
	set(value: number | string) {
		emits('update:modelValue', value);
	}
});

const debouncedSearch = refDebounced(proxyValue, 250)

const filteredMailList = computed(() => {
	let output: any[] = []

	const searchValue = debouncedSearch.value?.trim()
	if (!searchValue) {
		output = props.items
	} else {
		output = props.items.filter((item) => {
			return item.title.includes(debouncedSearch.value)
				|| item.url.includes(debouncedSearch.value)
				|| item.description.includes(debouncedSearch.value)
		})
	}

	return output
})

function getBadgeVariantFromLabel(label: string) {
	if (['work'].includes(label.toLowerCase()))
		return 'default'

	if (['personal'].includes(label.toLowerCase()))
		return 'outline'

	return 'secondary'
}

function extractTwitterUsername(url: string): string | null {
	const regex = /(?:https?:\/\/)?(?:www\.)?twitter\.com\/([a-zA-Z0-9_]+)/;

	const match = url.match(regex);

	if (match && match.length > 1) {
		return match[1];
	} else {
		return null;
	}
}

const pagination = ref({
	pageIndex: 0,
	pageSize: 10,
})

const table = {
	get data() { return props.items },
	data2() { return props.items },
	getState: () => ({
		pagination: {
			pageIndex: props.params.page,
			pageSize: props.params.perPage,
		},
	}),
	getPageCount: () => 1,
	getCanPreviousPage: () => true,
	getCanNextPage: () => true,
	setPageIndex: (index: number) => {
		pagination.value.pageIndex = index
		emits('setPageIndex', pagination.value.pageIndex)
	},
	previousPage: () => {
		pagination.value.pageIndex--
		emits('previousPage', pagination.value.pageIndex)
	},
	nextPage: () => {
		pagination.value.pageIndex++
		emits('nextPage', pagination.value.pageIndex)
	},
	setPageSize: (size: number) => {
		pagination.value.pageSize = size
		emits('perPage', size)
	},
}
</script>

<style scoped>
.list-enter-active,
.list-leave-active {
	transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
	opacity: 0;
	transform: translateX(30px);
}
</style>
