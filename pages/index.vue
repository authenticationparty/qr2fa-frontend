<template>
	<div class="flex justify-center items-center h-screen w-screen">
		<div
			class=" text-center border-gray-200 border-1 shadow px-8 py-4 rounded-md w-4/12"
		>
			<div class="flex justify-center">
				<QR2FALogo height="128" width="128" />
			</div>
			<p class="text-2xl">Welcome to
				<a class="font-semibold text-blue-500 hover:text-blue-600 duration-300" href="https://github.com/qr2fa">
					QR2FA
				</a>
			</p>
			<p class="text-gray-500">A pretty insecure way to authenticate</p>
			<hr class="w-full border-gray-400 my-2" />
			<div class="flex gap-3">
				<NuxtLink
					to='/login'
					class="rounded w-1/2 py-1 text-white bg-green-400 hover:bg-green-500 duration-300">
					Log In
				</NuxtLink>
				<NuxtLink
					to='/register'
					class="rounded w-1/2 py-1 text-white bg-green-400 hover:bg-green-500 duration-300">
					Register
				</NuxtLink>
			</div>
		</div>
	</div>
</template>

<script lang="ts">
import Vue from 'vue'
export default Vue.extend({})
</script>
