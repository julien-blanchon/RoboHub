<script lang="ts">
	import type { RemoteCompute } from "$lib/elements/compute//RemoteCompute.svelte";
	import { ICON } from "$lib/utils/icon";
	import { BaseStatusBox, StatusHeader, StatusContent, StatusIndicator, StatusButton } from "$lib/components/3d/ui";

	interface Props {
		compute: RemoteCompute;
		handleClick?: () => void;
	}

	let { compute, handleClick }: Props = $props();

	// Output theme color (blue)
	const outputColor = "rgb(59, 130, 246)";
</script>

<!--
@component
Robot output box showing the status of robot joint commands output from AI sessions.
Displays robot command output information when session exists or connection prompt when disconnected.
-->

<BaseStatusBox 
	color={outputColor}
	borderOpacity={0.6}
	backgroundOpacity={0.2}
	opacity={compute.hasSession && compute.isRunning ? 1 : !compute.hasSession ? 0.4 : 0.6}
	onclick={handleClick}
>
	{#if compute.hasSession && compute.outputConnections}
		<!-- Active Robot Output State -->
		<StatusHeader 
			icon={ICON["icon-[ix--robotic-arm]"].svg} 
			text="COMMANDS" 
			color={outputColor}
			opacity={0.9}
		/>

		<StatusContent 
			title={compute.isRunning ? "AI Commands Active" : "Session Ready"}
			subtitle="Motor Control"
			color="rgb(191, 219, 254)"
			variant="primary"
		/>

		<!-- Status indicator based on running state -->
		{#if compute.isRunning}
			<!-- Active pulse indicator -->
			<StatusIndicator color={outputColor} type="pulse" />
		{:else}
			<!-- Ready but not running indicator -->
			<StatusIndicator color="rgb(245, 158, 11)" />
		{/if}
	{:else}
		<!-- No Session State -->
		<StatusHeader 
			icon={ICON["icon-[ix--robotic-arm]"].svg} 
			text="NO OUTPUT" 
			color={outputColor}
			opacity={0.7}
		/>

		<StatusContent 
			title={!compute.hasSession ? 'Need Session' : 'Click to Configure'}
			color="rgb(147, 197, 253)"
			variant="secondary"
		/>

		<StatusButton 
			icon={ICON["icon-[ix--robotic-arm]"].svg} 
			text="Setup Output"
			color={outputColor}
			backgroundOpacity={0.1}
			textOpacity={0.7}
		/>
	{/if}
</BaseStatusBox> 