<template>
<MkStickyContainer>
	<template #header><MkPageHeader :actions="headerActions" :tabs="headerTabs"/></template>
	<MkSpacer :content-max="800" :margin-min="16" :margin-max="32">
		<FormSuspense :p="init">
			<FormInput v-model="title">
				<template #label>{{ $ts.title }}</template>
			</FormInput>

			<FormTextarea v-model="description" :max="500">
				<template #label>{{ $ts.description }}</template>
			</FormTextarea>

			<div class="">
				<div v-for="file in files" :key="file.id" class="wqugxsfx" :style="{ backgroundImage: file ? `url(${ file.thumbnailUrl })` : null }">
					<div class="name">{{ file.name }}</div>
					<button v-tooltip="$ts.remove" class="remove _button" @click="remove(file)"><i class="fas fa-times"></i></button>
				</div>
				<FormButton primary @click="selectFile"><i class="fas fa-plus"></i> {{ $ts.attachFile }}</FormButton>
			</div>

			<FormSwitch v-model="isSensitive">{{ $ts.markAsSensitive }}</FormSwitch>

			<FormButton v-if="postId" primary @click="save"><i class="fas fa-save"></i> {{ $ts.save }}</FormButton>
			<FormButton v-else primary @click="save"><i class="fas fa-save"></i> {{ $ts.publish }}</FormButton>

			<FormButton v-if="postId" danger @click="del"><i class="fas fa-trash-alt"></i> {{ $ts.delete }}</FormButton>
		</FormSuspense>
	</MkSpacer>
</MkStickyContainer>
</template>

<script lang="ts" setup>
import { computed, inject, watch } from 'vue';
import FormButton from '@/components/ui/button.vue';
import FormInput from '@/components/form/input.vue';
import FormTextarea from '@/components/form/textarea.vue';
import FormSwitch from '@/components/form/switch.vue';
import FormSuspense from '@/components/form/suspense.vue';
import { selectFiles } from '@/scripts/select-file';
import * as os from '@/os';
import { useRouter } from '@/router';
import { definePageMetadata } from '@/scripts/page-metadata';
import { i18n } from '@/i18n';

const router = useRouter();

const props = defineProps<{
	postId?: string;
}>();

let init = $ref(null);
let files = $ref([]);
let description = $ref(null);
let title = $ref(null);
let isSensitive = $ref(false);

function selectFile(evt) {
	selectFiles(evt.currentTarget ?? evt.target, null).then(selected => {
		files = files.concat(selected);
	});
}

function remove(file) {
	files = files.filter(f => f.id !== file.id);
}

async function save() {
	if (props.postId) {
		await os.apiWithDialog('gallery/posts/update', {
			postId: props.postId,
			title: title,
			description: description,
			fileIds: files.map(file => file.id),
			isSensitive: isSensitive,
		});
		router.push(`/gallery/${props.postId}`);
	} else {
		const created = await os.apiWithDialog('gallery/posts/create', {
			title: title,
			description: description,
			fileIds: files.map(file => file.id),
			isSensitive: isSensitive,
		});
		router.push(`/gallery/${created.id}`);
	}
}

async function del() {
	const { canceled } = await os.confirm({
		type: 'warning',
		text: i18n.ts.deleteConfirm,
	});
	if (canceled) return;
	await os.apiWithDialog('gallery/posts/delete', {
		postId: props.postId,
	});
	router.push('/gallery');
}

watch(() => props.postId, () => {
	init = () => props.postId ? os.api('gallery/posts/show', {
		postId: props.postId,
	}).then(post => {
		files = post.files;
		title = post.title;
		description = post.description;
		isSensitive = post.isSensitive;
	}) : Promise.resolve(null);
}, { immediate: true });

const headerActions = $computed(() => []);

const headerTabs = $computed(() => []);

definePageMetadata(computed(() => props.postId ? {
	title: i18n.ts.edit,
	icon: 'fas fa-pencil-alt',
} : {
	title: i18n.ts.postToGallery,
	icon: 'fas fa-pencil-alt',
}));
</script>

<style lang="scss" scoped>
.wqugxsfx {
	height: 200px;
	background-size: contain;
	background-position: center;
	background-repeat: no-repeat;
	position: relative;

	> .name {
		position: absolute;
		top: 8px;
		left: 9px;
		padding: 8px;
		background: var(--panel);
	}

	> .remove {
		position: absolute;
		top: 8px;
		right: 9px;
		padding: 8px;
		background: var(--panel);
	}
}
</style>
