

<script setup lang="ts">
    import type { FormError, FormSubmitEvent } from '#ui/types'
	import { ACCOUNT } from '~/libs/appwrite';
	import { useAuthStore } from '~/store/auth.store';


	defineProps({
		toggleLogin: {
			type: Function,
			required: true,
		},
	})

	const toast = useToast();
	const authStore = useAuthStore();
	const router = useRouter();

	const isLoading = ref(false);
	const error = ref('');

	const state = reactive({
		email: undefined,
		password: undefined,
	})

	const validate = (state: any): FormError[] => {
		const errors = []
		if (!state.email) errors.push({ path: 'email', message: 'Email is requierd' })
		if (!state.password) errors.push({ path: 'password', message: 'Password is required' })
		return errors
	}

	async function onSubmit(event: FormSubmitEvent<any>) {
		isLoading.value = true;
		const { email, password } = event.data

		try{
			await ACCOUNT.createEmailSession(email, password);
			const response = await ACCOUNT.get(); 
			
			authStore.set({
				email: response.email,
				id: response.$id,
				name: response.name,
				status: response.status,
			})
			toast.add({
				title: "Logged in",
				description: "You are now logged in"
			})
			await router.push('/')
		}catch(e: any){
			error.value = e.message;
			isLoading.value = false;
		}
	}
</script>

<template>
	<UAlert
		icon="i-heroicons-command-line"
    	:description="error"
    	title="Error"
		v-if="error"
		color="red"
		variant="outline"
  	/>
	<UForm
		:validate="validate"
		:state="state"
		class="space-y-4"
		@submit="onSubmit"
	>
		<UFormGroup label="Email" name="email">
			<UInput v-model="state.email" color="blue" size="lg" />
		</UFormGroup>

		<UFormGroup label="Password" name="password">
			<UInput v-model="state.password" type="password" color="blue" size="lg" />
		</UFormGroup>

		<div class="text-sm text-neutral-500">
			Not registered yet?
			<span class="text-blue-500 hover:underline" role="button" @click="$props.toggleLogin">
				Sign up
			</span>
		</div>

		<UButton type="submit" color="blue" class="w-full" block size="lg" :disabled="isLoading">
			<template v-if="isLoading">
				<Icon name="svg-spinners:3-dots-fade" class="w-5 h-5" />
			</template>
			<template v-else>Next</template>
		</UButton>
	</UForm>
</template>



