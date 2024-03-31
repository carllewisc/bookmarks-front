<script setup lang="ts">
import { h } from 'vue'
import { useForm } from 'vee-validate'
import { toTypedSchema } from '@vee-validate/zod'
import * as z from 'zod'
import { useQueryClient, useMutation } from '@tanstack/vue-query';

import { Input } from '@/components/ui/input'
import { FormControl, FormDescription, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form'
import { Separator } from '@/components/ui/separator'
import { Button } from '@/components/ui/button'
import { toast } from '@/components/ui/toast'

const profileFormSchema = toTypedSchema(z.object({
	url: z.string().url({ message: 'Please enter a valid URL.' }),
}))

const { handleSubmit, resetForm, meta } = useForm({
	validationSchema: profileFormSchema,
	initialValues: {
		url: 'https://twitter.com/shadcn'
	},
})

const createBookmark = async (payload:  any) => {
	try {
		const data = await fetch('http://localhost:8000/bookmarks', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json',
			},
			body: JSON.stringify(payload),
		})

		if (!data.ok) {
			throw new Error('Failed to create bookmark');
		}

		return data.json();
	} catch (error: any) {
		throw error;
	}
}

const queryClient = useQueryClient();

const { mutate: mutateCreateFile } = useMutation({
	mutationKey: ["create-bookmark"],
	mutationFn: (payload) => createBookmark(payload),
	onError: (error) => {
		toast({
			title: 'Error',
			description: error.message,
		});
	},
});

const doCreateBookmark = async (payload: any) => {
	mutateCreateFile({
		...payload,
		title: 'Shadcn',
		description: `Hi, I'm Shadcn. I'm a software engineer and I love to build things.`,
	}, {
		onSuccess: () => {
			toast({
				duration: 1000,
				title: 'You submitted the following values:',
				description: h('pre', { class: 'mt-2 w-[340px] rounded-md bg-slate-950 p-4' }, h('code', { class: 'text-white' }, JSON.stringify(payload, null, 2))),
			})

			queryClient.invalidateQueries({ queryKey: ['bookmarks'] });
		},
	});
};

const onSubmit = handleSubmit((values) => {
	if (meta.value.valid) {
		doCreateBookmark(values)
	} else {
		toast({
			title: 'Error',
			description: 'Please fix the errors in the form.',
		})
	}
});
</script>

<template>
  <div>
    <div>
      <h3 class="text-lg font-medium">
        Add bookmark
      </h3>
      <p class="text-sm text-muted-foreground">
        Add a new bookmark to your collection.
      </p>
    </div>
    <Separator class="my-2" />
    <form
      class="space-y-8"
      @submit="onSubmit"
    >
      <FormField
        v-slot="{ componentField }"
        name="url"
      >
        <FormItem>
          <FormLabel>
            URL
          </FormLabel>
          <FormDescription>
            Add a link to your website, blog, or social media profile.
          </FormDescription>
          <FormControl>
            <Input
              type="url"
              v-bind="componentField"
            />
          </FormControl>
          <FormMessage />
        </FormItem>
      </FormField>

      <div class="flex gap-2 justify-start">
        <Button type="submit">
          Add bookmark
        </Button>

        <Button
          type="button"
          variant="outline"
          @click="resetForm"
        >
          Reset form
        </Button>
      </div>
    </form>
  </div>
</template>
