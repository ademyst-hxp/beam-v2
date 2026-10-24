<script setup lang="ts">
import axios from "axios";

import { countryList } from "@/utils/countries";
import { isYoungerThan18 } from "@/utils/profiles";
import { useSession } from "~/stores/session";

import { AtSymbolIcon, CheckIcon, XMarkIcon } from "@heroicons/vue/24/solid";

import Button from "~/components/Button.vue";

useHead({
	title: "Rejoindre Beam",
	meta: [{ name: "description", content: "Créer un compte." }],
});

const { $client, $apiUrl } = useNuxtApp();
const { refreshSession } = useSession();
$client.refresh();

const router = useRouter();

const step = ref<number>(0);

const payload = ref<Record<string, string | boolean>>({
	email: "",
	password: "",
	username: "",
	display_name: "",
	birthday: new Date().toISOString().split("T")[0]!,
	country: "FR",
	private: true,
	sudo_code: "",
});

const valid = ref<Record<string, boolean>>({
	email: true,
	password: true,
	username: true,
	display_name: true,
	birthday: true,
	country: true,
	private: true,
	sudo_code: true,
});

const pwSpecs = {
	length: false,
	uppercase: false,
	lowercase: false,
	digit: false,
	special: false,
};

async function signup() {
	const res = await axios.post(`${$apiUrl}/auth/signup`, payload.value);

	if (res.status == 200) {
		const session = await $client.login(
			payload.value!.username as string,
			payload.value!.password as string,
		);
		await $client.refresh(session?.token!);

		router.push("/@" + payload.value.username);
	} else {
		console.error(res.status, res.data);
	}
}

onMounted(async () => {
	await refreshSession();
});

function check(field: string, regex: RegExp, min?: number, max?: number) {
	let val = payload.value[field as string] as string;

	if (min && val.length < min) {
		valid.value[field] = false;
		return false;
	}

	if (!regex.test(val)) {
		valid.value[field] = false;
		return false;
	}

	if (max && val.length > max) {
		valid.value[field] = false;
		return false;
	}

	valid.value[field] = true;
	return true;
}

function checkPassword() {
	const val = payload.value.password as string;

	const length = val.length >= 8;
	const uppercase = /[A-Z]/.test(val);
	const lowercase = /[a-z]/.test(val);
	const digit = /[0-9]/.test(val);
	const special = /[!@#$%^&*(),.?":{}|<>]/.test(val);

	pwSpecs.length = length;
	pwSpecs.uppercase = uppercase;
	pwSpecs.lowercase = lowercase;
	pwSpecs.digit = digit;
	pwSpecs.special = special;

	valid.value.password = length && uppercase && lowercase && digit && special;
	return valid.value.password;
}
</script>
<template>
	<main class="p-4 xs:p-8">
		<nav class="flex max-w-lg px-10 mx-auto mb-1">
			<a href="/" class="text-primary font-semibold">
				Retour à l'accueil
			</a>
			<div class="grow"></div>
			<a href="/auth/login" class="text-primary font-semibold">
				Se connecter
			</a>
		</nav>
		<section
			class="bg-surface text-on-surface border border-surface-border rounded-4xl max-w-lg p-8 mx-auto space-y-8"
		>
			<h1 class="text-2xl font-bold mb-4">
				La création de compte est fermée.
			</h1>
			<p class="max-w-lg mx-auto">
				Beam s'apprête à rendre l'âme au profit d'un nouveau projet bien
				plus dynamique et moderne. Nous avons donc décidé de fermer la
				création de compte sur cette version de Beam afin de nous
				concentrer sur le développement du nouveau projet.
			</p>
			<div class="flex flex-col items-center justify-center gap-2">
				<a
					href="https://ademyst.ejnalo.me"
					class="text-primary text-xl font-semibold hover:underline"
				>
					Aller sur Ademyst
				</a>
				<Button
					label="Retour"
					type="transparent"
					:handler="() => router.back()"
				/>
			</div>
		</section>
	</main>
</template>
