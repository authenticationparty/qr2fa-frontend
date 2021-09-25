<template>
	<div class="flex justify-center items-center h-screen w-screen">
		<div
			class="text-center border-gray-200 border-1 shadow px-8 py-4 rounded-md"
		>
			<div class="flex justify-center">
				<QR2FALogo height="128" width="128" />
			</div>
			<p>Log In</p>
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
					Log In
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
import Cookies from 'js-cookie'
import Joi from 'joi';

// User schema
const UserSchema = Joi.object({
	username: Joi
		.string().alphanum().required().label('Username')
		.min(4).max(64),
	password: Joi
		.string().alphanum().required().label('Password')
		.min(6).max(128),
})

export default Vue.extend({
    data(): any {
        return {
            inputComponents: true,
			username: "",
			password: "",
        };
    },
    methods: {
        async toggleQrCode(): Promise<void> {
			// Check if the data is valid
			const UserValidation = UserSchema.validate({
				username: this.username,
				password: this.password,
			});
			if (UserValidation?.error) {
				return this.$toast.show({
					type: 'danger',
					title: 'Error',
					message: UserValidation.error.message,
				})
			}

			const authentication = await fetch(
					`http://${process.env.apiUrl}:${process.env.apiPort}/authenticate`,
					{
						method: 'POST',
						headers: {
							'Content-Type': 'application/json',
						},
						body: JSON.stringify(UserValidation.value)
					}
				).then(_=>_.json());

			if (!authentication?.success) {
				return this.$toast.show({
							type: 'danger',
							title: 'Unable to log in',
							message: authentication?.message,
						})
			}

            this.inputComponents = !this.inputComponents;
			this.$nextTick(()=>{
				const qrElem = this.$refs.qrcode
				QRCode.toCanvas(
					qrElem,
					`http://${process.env.apiUrl}:${process.env.apiPort}/acceptLogin?u=${this.username}`,
					(err) => {
						if (err) return this.$toast.show({
							type: 'danger',
							title: 'Server error',
							message: 'Couldnt render QR',
						})
					}
				)

				// Meter algo onda un hash en el server, que si no esta ese hash,
				// la pass no es válida y no abre la WebSocket porque no es válido el hash!!!

				// Es decir, en vez de hacer el hash aca, q lo haga el server
				// Y si no está el hash en la session, sacarlo a la mierda

				const socket = new WebSocket(`ws://${process.env.apiUrl}:${Number(process.env.apiPort)+1}`);
				socket.addEventListener('open', function (_event) {
					socket.send(`hash:${authentication.Hash}`)
				});

				socket.addEventListener('message', function(_event) {
					if (_event?.data.startsWith('.logged')) {
						Cookies.set('qr2fa-SessId', _event?.data.replace('.logged', ''), {
							expires: 1,
						})
						location.href = '/superSecretPage';
					}
				});
			})
        }
    }
})
</script>
