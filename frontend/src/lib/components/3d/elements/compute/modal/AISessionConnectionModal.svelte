<script lang="ts">
	import * as Dialog from "@/components/ui/dialog";
	import { Button } from "@/components/ui/button";
	import * as Card from "@/components/ui/card";
	import { Badge } from "@/components/ui/badge";
	import { Input } from "@/components/ui/input";
	import { Label } from "@/components/ui/label";
	import * as Alert from "@/components/ui/alert";
	import { remoteComputeManager } from "$lib/elements/compute//RemoteComputeManager.svelte";
	import type { RemoteCompute } from "$lib/elements/compute//RemoteCompute.svelte";
	import type { AISessionConfig } from "$lib/elements/compute//RemoteComputeManager.svelte";
	import { settings } from "$lib/runes/settings.svelte";
	import { toast } from "svelte-sonner";

	interface Props {
		workspaceId: string;
		open: boolean;
		compute: RemoteCompute;
	}

	let { open = $bindable(), compute, workspaceId }: Props = $props();

	let isConnecting = $state(false);
	let sessionId = $state('');
	let policyPath = $state('./checkpoints/act_so101_beyond');
	let cameraNames = $state('front');
	let useProvidedWorkspace = $state(false);

	// Auto-generate session ID when modal opens
	$effect(() => {
		if (open && compute && !sessionId) {
			sessionId = `${compute.id}-session-${Date.now()}`;
		}
	});

	async function handleCreateSession() {
		if (!compute) return;

		if (!sessionId.trim() || !policyPath.trim()) {
			toast.error('Please fill in all required fields');
			return;
		}

		isConnecting = true;
		try {
			const cameras = cameraNames.split(',').map(name => name.trim()).filter(name => name);
			if (cameras.length === 0) {
				cameras.push('front');
			}

			const config: AISessionConfig = {
				sessionId: sessionId.trim(),
				policyPath: policyPath.trim(),
				cameraNames: cameras,
				transportServerUrl: settings.transportServerUrl,
				workspaceId: useProvidedWorkspace ? workspaceId : undefined
			};

			const result = await remoteComputeManager.createSession(compute.id, config);
			if (result.success) {
				toast.success(`AI session created: ${sessionId}`);
				open = false;
			} else {
				toast.error(`Failed to create session: ${result.error}`);
			}
		} catch (error) {
			console.error('Session creation error:', error);
			toast.error('Failed to create session');
		} finally {
			isConnecting = false;
		}
	}

	async function handleStartSession() {
		if (!compute) return;

		isConnecting = true;
		try {
			const result = await remoteComputeManager.startSession(compute.id);
			if (result.success) {
				toast.success('AI session started');
			} else {
				toast.error(`Failed to start session: ${result.error}`);
			}
		} catch (error) {
			console.error('Session start error:', error);
			toast.error('Failed to start session');
		} finally {
			isConnecting = false;
		}
	}

	async function handleStopSession() {
		if (!compute) return;

		isConnecting = true;
		try {
			const result = await remoteComputeManager.stopSession(compute.id);
			if (result.success) {
				toast.success('AI session stopped');
			} else {
				toast.error(`Failed to stop session: ${result.error}`);
			}
		} catch (error) {
			console.error('Session stop error:', error);
			toast.error('Failed to stop session');
		} finally {
			isConnecting = false;
		}
	}

	async function handleDeleteSession() {
		if (!compute) return;

		isConnecting = true;
		try {
			const result = await remoteComputeManager.deleteSession(compute.id);
			if (result.success) {
				toast.success('AI session deleted');
			} else {
				toast.error(`Failed to delete session: ${result.error}`);
			}
		} catch (error) {
			console.error('Session delete error:', error);
			toast.error('Failed to delete session');
		} finally {
			isConnecting = false;
		}
	}
</script>

<Dialog.Root bind:open>
	<Dialog.Content
		class="max-h-[80vh] max-w-2xl overflow-y-auto border-slate-600 bg-slate-900 text-slate-100"
	>
		<Dialog.Header class="pb-3">
			<Dialog.Title class="flex items-center gap-2 text-lg font-bold text-slate-100">
				<span class="icon-[mdi--robot-outline] size-5 text-purple-400"></span>
				AI Compute Session - {compute.name || 'No Compute Selected'}
			</Dialog.Title>
			<Dialog.Description class="text-sm text-slate-400">
				Configure and manage ACT model inference sessions for robot control
			</Dialog.Description>
		</Dialog.Header>

		<div class="space-y-4">
			<!-- Current Session Status -->
			<div
				class="flex items-center justify-between rounded-lg border border-purple-500/30 bg-purple-900/20 p-3"
			>
				<div class="flex items-center gap-2">
					<span class="icon-[mdi--brain] size-4 text-purple-400"></span>
					<span class="text-sm font-medium text-purple-300">Session Status</span>
				</div>
				{#if compute.hasSession}
					<Badge variant="default" class="bg-purple-600 text-xs">
						{compute.statusInfo.statusText}
					</Badge>
				{:else}
					<Badge variant="secondary" class="text-xs text-slate-400">No Session</Badge>
				{/if}
			</div>

			<!-- Current Session Details -->
			{#if compute.hasSession && compute.sessionData}
				<Card.Root class="border-purple-500/30 bg-purple-500/5">
					<Card.Header>
						<Card.Title class="flex items-center gap-2 text-base text-purple-200">
							<span class="icon-[mdi--cog] size-4"></span>
							Current Session
						</Card.Title>
					</Card.Header>
					<Card.Content>
						<div class="space-y-3">
							<div class="rounded-lg border border-purple-500/30 bg-purple-900/20 p-3">
								<div class="grid grid-cols-2 gap-2 text-xs">
									<div>
										<span class="text-purple-300 font-medium">Session ID:</span>
										<span class="text-purple-100 block">{compute.sessionId}</span>
									</div>
									<div>
										<span class="text-purple-300 font-medium">Status:</span>
										<span class="text-purple-100 block">{compute.statusInfo.emoji} {compute.statusInfo.statusText}</span>
									</div>
									<div>
										<span class="text-purple-300 font-medium">Policy:</span>
										<span class="text-purple-100 block">{compute.sessionConfig?.policyPath}</span>
									</div>
									<div>
										<span class="text-purple-300 font-medium">Cameras:</span>
										<span class="text-purple-100 block">{compute.sessionConfig?.cameraNames.join(', ')}</span>
									</div>
								</div>
							</div>

							<!-- Connection Details -->
							<div class="rounded-lg border border-green-500/30 bg-green-900/20 p-3">
								<div class="text-sm font-medium text-green-300 mb-2">📡 Inference Server Connections</div>
								<div class="space-y-1 text-xs">
									<div>
										<span class="text-green-400">Workspace:</span>
										<span class="text-green-200 font-mono ml-2">{compute.sessionData.workspace_id}</span>
									</div>
									{#each Object.entries(compute.sessionData.camera_room_ids) as [camera, roomId]}
										<div>
											<span class="text-green-400">📹 {camera}:</span>
											<span class="text-green-200 font-mono ml-2">{roomId}</span>
										</div>
									{/each}
									<div>
										<span class="text-green-400">📥 Joint Input:</span>
										<span class="text-green-200 font-mono ml-2">{compute.sessionData.joint_input_room_id}</span>
									</div>
									<div>
										<span class="text-green-400">📤 Joint Output:</span>
										<span class="text-green-200 font-mono ml-2">{compute.sessionData.joint_output_room_id}</span>
									</div>
								</div>
							</div>

							<!-- Session Controls -->
							<div class="flex gap-2">
								{#if compute.canStart}
									<Button
										variant="default"
										size="sm"
										onclick={handleStartSession}
										disabled={isConnecting}
										class="bg-green-600 hover:bg-green-700 text-xs disabled:opacity-50"
									>
										{#if isConnecting}
											<span class="icon-[mdi--loading] animate-spin mr-1 size-3"></span>
											Starting...
										{:else}
											<span class="icon-[mdi--play] mr-1 size-3"></span>
											Start Inference
										{/if}
									</Button>
								{/if}
								{#if compute.canStop}
									<Button
										variant="secondary"
										size="sm"
										onclick={handleStopSession}
										disabled={isConnecting}
										class="text-xs disabled:opacity-50"
									>
										{#if isConnecting}
											<span class="icon-[mdi--loading] animate-spin mr-1 size-3"></span>
											Stopping...
										{:else}
											<span class="icon-[mdi--stop] mr-1 size-3"></span>
											Stop Inference
										{/if}
									</Button>
								{/if}
								<Button
									variant="destructive"
									size="sm"
									onclick={handleDeleteSession}
									disabled={isConnecting}
									class="text-xs disabled:opacity-50"
								>
									{#if isConnecting}
										<span class="icon-[mdi--loading] animate-spin mr-1 size-3"></span>
										Deleting...
									{:else}
										<span class="icon-[mdi--delete] mr-1 size-3"></span>
										Delete Session
									{/if}
								</Button>
							</div>
						</div>
					</Card.Content>
				</Card.Root>
			{/if}

			<!-- Create New Session -->
			{#if !compute.hasSession}
				<Card.Root class="border-purple-500/30 bg-purple-500/5">
					<Card.Header>
						<Card.Title class="flex items-center gap-2 text-base text-purple-200">
							<span class="icon-[mdi--plus-circle] size-4"></span>
							Create AI Session
						</Card.Title>
					</Card.Header>
					<Card.Content>
						<div class="space-y-4">
							<div class="grid grid-cols-2 gap-4">
								<div class="space-y-2">
									<Label for="sessionId" class="text-purple-300">Session ID</Label>
									<Input
										id="sessionId"
										bind:value={sessionId}
										placeholder="my-session-01"
										class="bg-slate-800 border-slate-600 text-slate-100"
									/>
								</div>
								<div class="space-y-2">
									<Label for="policyPath" class="text-purple-300">Policy Path</Label>
									<Input
										id="policyPath"
										bind:value={policyPath}
										placeholder="./checkpoints/act_so101_beyond"
										class="bg-slate-800 border-slate-600 text-slate-100"
									/>
								</div>
							</div>

							<div class="grid grid-cols-2 gap-4">
								<div class="space-y-2">
									<Label for="cameraNames" class="text-purple-300">Camera Names</Label>
									<Input
										id="cameraNames"
										bind:value={cameraNames}
										placeholder="front, wrist, overhead"
										class="bg-slate-800 border-slate-600 text-slate-100"
									/>
									<p class="text-xs text-slate-400">Comma-separated camera names</p>
								</div>
								<div class="space-y-2">
									<Label for="transportServerUrl" class="text-purple-300">Transport Server URL</Label>
									<Input
										id="transportServerUrl"
										value={settings.transportServerUrl}
										disabled
										placeholder="http://localhost:8000"
										class="bg-slate-800 border-slate-600 text-slate-100 opacity-60 cursor-not-allowed"
										title="Change this value in the settings panel"
									/>
									<p class="text-xs text-slate-400">Configure in settings panel</p>
								</div>
							</div>

							<div class="flex items-center space-x-2">
								<input
									type="checkbox"
									id="useWorkspace"
									bind:checked={useProvidedWorkspace}
									class="rounded border-slate-600 bg-slate-800"
								/>
								<Label for="useWorkspace" class="text-purple-300 text-sm">
									Use current workspace ({workspaceId})
								</Label>
							</div>

							<Alert.Root>
								<span class="icon-[mdi--information] size-4"></span>
								<Alert.Description>
									This will create a new ACT inference session with dedicated rooms for camera inputs,
									joint inputs, and joint outputs in the inference server communication system.
								</Alert.Description>
							</Alert.Root>

							<Button
								variant="default"
								onclick={handleCreateSession}
								disabled={isConnecting || !sessionId.trim() || !policyPath.trim()}
								class="w-full bg-purple-600 hover:bg-purple-700 disabled:opacity-50"
							>
								{#if isConnecting}
									<span class="icon-[mdi--loading] animate-spin mr-2 size-4"></span>
									Creating Session...
								{:else}
									<span class="icon-[mdi--rocket-launch] mr-2 size-4"></span>
									Create AI Session
								{/if}
							</Button>
						</div>
					</Card.Content>
				</Card.Root>
			{/if}

			<!-- Quick Info -->
			<div class="rounded border border-slate-700 bg-slate-800/30 p-2 text-xs text-slate-500">
				<span class="icon-[mdi--information] mr-1 size-3"></span>
				AI sessions require a trained ACT model and create dedicated communication rooms for video inputs,
				robot joint states, and control outputs in the inference server system.
			</div>
		</div>
	</Dialog.Content>
</Dialog.Root> 