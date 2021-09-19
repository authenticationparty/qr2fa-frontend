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
				<br class="my-2" />
				<input
					ref="email"
					v-model="email"
					type="text"
					class="bg-gray-100 rounded px-4 py-1"
					placeholder="Email"
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
const UserSchema = Joi.object({
	username: Joi
		.string().alphanum().required().label('Username')
		.min(4).max(64),
	password: Joi
		.string().alphanum().required().label('Password')
		.min(6).max(128),
	email: Joi
		.string().email({
			tlds: {
				allow: false,
			}
		}).required().label('Email')
		.max(512),
})

export default Vue.extend({
    data() {
        return {
            inputComponents: true,
			username: "",
			password: "",
			email: "", // Asking for email so user can reset device if lost
        };
    },
    methods: {
        async toggleQrCode() {
			// Check if the data is valid
			const UserValidation = UserSchema.validate({
				username: this.username,
				password: this.password,
				email: this.email,
			});
			if (UserValidation?.error) {
				return this.$toast.show({
					type: 'danger',
					title: 'Error',
					message: UserValidation.error.message,
				})
			}

			// Validate if username/email is available
			const avCheck = await fetch(process.env.apiUrl + '/isAvailable', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json',
				},
				body: JSON.stringify(UserValidation.value),
			}).then(_=>_.json());
			if (!avCheck.success || !avCheck.available) {
				return this.$toast.show({
							type: 'danger',
							title: 'Error',
							message: avCheck.message,
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
