<template>
	<div>
		<p>
			You shouldn't be able to see this page nor your username<br>
			if you are not logged in
		</p>

		<br>
		<div class="flex">
			Your username is&nbsp;
			<div v-if="username">
				<code>{{ username }}</code>
			</div>
			<div v-else>
					<svg class="animate-spin ml-2 h-5 w-5 text-black" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
						<circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
						<path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
					</svg>
			</div>
		</div>
	</div>
</template>
<script lang="ts">
import Vue from 'vue'
import Cookies from 'js-cookie'

const SessId = Cookies.get('qr2fa-SessId');

async function getUsername(): Promise<string> {
	if (SessId) {
		const User = await fetch(
			`http://${process.env.apiUrl}:${process.env.apiPort}/getUser`,
			{
				method: 'POST',
				headers: {
					'Content-Type': 'application/json',
				},
				body: JSON.stringify({
					SessId,
				})
			}
		).then(_=>_.json());
		return User.data.username;
	} else
		return '';
}

export default Vue.extend({
	data(): any {
        return {
            username: '',
        };
    },
	beforeMount(): any {
		if (!SessId) {
			location.href = '/login';
			return;
		}
		(async ()=> {
			this.username = await getUsername();
		})();
	}
})
</script>
