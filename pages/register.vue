<template>
	<div class="flex justify-center items-center h-screen w-screen">
		<div
			class="text-center border-gray-200 border-1 shadow px-8 py-4 rounded-md"
		>
			<h1 class="font-bold text-3xl">qr2fa</h1>
			<p>Register</p>
			<hr class="w-full border-gray-400 my-2" />
			<div v-if="inputComponents">
				<input
					ref="username"
					v-model="username"
					type="text"
					class="bg-gray-100 rounded px-4 py-1"
					placeholder="Username"
				/>
				<br class="my-2" />
				<input
					ref="password"
					v-model="password"
					type="password"
					class="bg-gray-100 rounded px-4 py-1"
					placeholder="Password"
				/>

				<br />
				<button
					class="bg-green-500 hover:bg-green-600 rounded w-full py-1 text-white mt-2 duration-300"
					@click="toggleQrCode()"
				>
					Register
				</button>
			</div>
			<div v-else>
				<p>Hey <b>{{ username }}</b></p>
				<p class="text-gray-600">Scan this QR Code to continue</p>
				<div class="flex justify-center">
					<canvas id="qrcode" ref="qrcode"></canvas>
				</div>
			</div>
		</div>
	</div>
</template>

<script lang="ts">
import Vue from 'vue'
import QRCode from 'qrcode';
import Joi from 'joi';

// User schema
const UsernameSchema = Joi
	.string().alphanum().required().label('Username')
	.min(4).max(64);
const PasswordSchema = Joi
	.string().alphanum().required().label('Password')
	.min(6).max(128);

export default Vue.extend({
    data() {
        return {
            inputComponents: true,
			username: "",
			password: "",
        };
    },
    methods: {
        toggleQrCode() {
			const UsernameValidation = UsernameSchema.validate(this.username);
			if (UsernameValidation?.error) {
				return this.$toast.show({
					type: 'danger',
					title: 'Error',
					message: UsernameValidation.error.message,
				})
			}
			const PasswordValidation = PasswordSchema.validate(this.password);
			if (PasswordValidation?.error) {
				return this.$toast.show({
					type: 'danger',
					title: 'Error',
					message: PasswordValidation.error.message,
				})
			}

            this.inputComponents = !this.inputComponents;
			this.$nextTick(()=>{
				const qrElem = this.$refs.qrcode
				QRCode.toCanvas(
					qrElem,
					`http://192.168.0.108:8080/register?u=${this.username}&p=${this.password}`,
					(err) => {
						if (err) return this.$toast.show({
							type: 'danger',
							title: 'Server error',
							message: 'Couldnt render QR',
						})
					}
				)
			})
        }
    }
})
</script>
