<script lang="ts">
	import ClassroomManagement from '$lib/components/ClassroomManagement.svelte';
	import { page } from '$app/stores';

	const classId = $derived($page.params.id);
</script>

<ClassroomManagement {classId} />
