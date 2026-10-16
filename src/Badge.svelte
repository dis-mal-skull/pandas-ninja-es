<script lang="ts">
	const COLORS = {
		easy: "bg-green-500",
		medium: "bg-orange-500",
		hard: "bg-red-500",
	};

	const LABELS = {
		easy: "Fácil",
		medium: "Medio",
		hard: "Difícil",
	};

	export let level: string;

	$: bgColor = COLORS[level];
	$: label = LABELS[level] ?? level;
</script>

<div class="text-xs {bgColor} text-white rounded  px-2 py-1 h-fit text-center">
	{label}
</div>
