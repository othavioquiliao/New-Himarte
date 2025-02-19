<script lang="ts">
	import Button from '$lib/components/ui/button/button.svelte';
	import Separator from '$lib/components/ui/separator/separator.svelte';
	import { CheckCircle2 } from 'lucide-svelte';
	import { toast } from 'svelte-sonner';
	import { enhance } from '$app/forms';
	import StepOne from '$lib/components/Cadastro/StepOne.svelte';
	import StepTwo from '$lib/components/Cadastro/StepTwo.svelte';
	import StepThree from '$lib/components/Cadastro/StepThree.svelte';
	import type { FormData } from '$lib/components/Cadastro/types';

	// Estado do formulário usando $state
	let currentStep = $state(1);
	let formData = $state<FormData>({
		// Dados pessoais (Step 1)
		fullName: '',
		genero: '',
		dataNascimento: '',
		telefone: '',
		emailCliente: '',

		// Endereço (Step 2)
		cep: '',
		cidade: '',
		rua: '',
		bairro: '',
		estado: '',
		numero: '',
		complemento: '',
		pontoReferencia: '',

		// Plano (Step 3)
		planoNome: '',
		planoMegas: '',
		valorPlano: '',
		planoModelo: 'CPF',
		cpf: '',
		cnpj: '',
		promoCode: ''
	});

	// Validação por etapa
	const stepValidation = {
		1: (data: FormData) => {
			return (
				data.fullName && data.genero && data.dataNascimento && data.telefone && data.emailCliente
			);
		},
		2: (data: FormData) => {
			return data.cep && data.cidade && data.rua && data.bairro && data.estado && data.numero;
		},
		3: (data: FormData) => {
			return (
				data.planoNome &&
				data.planoMegas &&
				data.planoModelo &&
				((data.planoModelo === 'CPF' && data.cpf) || (data.planoModelo === 'CNPJ' && data.cnpj))
			);
		}
	};

	// Função para navegar entre as etapas
	function navigateStep(direction: 'next' | 'prev') {
		if (direction === 'next' && currentStep < 3) {
			if (stepValidation[currentStep as keyof typeof stepValidation](formData)) {
				currentStep++;
			} else {
				toast.error('Por favor, preencha todos os campos obrigatórios');
			}
		} else if (direction === 'prev' && currentStep > 1) {
			currentStep--;
		}
	}

	let isSubmitting = $state(false);
</script>

<div class="min-h-full w-full flex items-center justify-center pt-28">
	<form
		method="POST"
		action="?/novoCadastro"
		class="w-fit flex h-full flex-col gap-7"
		use:enhance={({ formData: submitFormData, cancel }) => {
			// Validação antes de enviar
			if (!stepValidation[3](formData)) {
				toast.error('Por favor, preencha todos os campos obrigatórios');
				cancel();
				return;
			}

			isSubmitting = true;
			toast.loading('Enviando cadastro...');

			// Adiciona os dados do formData ao submitFormData
			Object.entries(formData).forEach(([key, value]) => {
				if (value) {
					submitFormData.set(key, value.toString());
				}
			});

			return async ({ result }) => {
				isSubmitting = false;

				if (result.type === 'success') {
					toast.success('Cadastro realizado com sucesso!');
					// Limpa o formulário
					formData = {
						fullName: '',
						genero: '',
						dataNascimento: '',
						telefone: '',
						emailCliente: '',
						cep: '',
						cidade: '',
						rua: '',
						bairro: '',
						estado: '',
						numero: '',
						complemento: '',
						pontoReferencia: '',
						planoNome: '',
						planoMegas: '',
						valorPlano: '',
						planoModelo: 'CPF',
						cpf: '',
						cnpj: '',
						promoCode: ''
					};
					currentStep = 1;
				} else if (result.type === 'failure') {
					toast.error(result.data?.mensagem || 'Erro ao processar o cadastro');
				} else {
					toast.error('Erro ao processar o cadastro');
				}
			};
		}}
	>
		<div class="flex flex-col gap-1 items-center">
			<h1 class="font-bold text-center text-4xl">Quase lá! Complete seu cadastro</h1>
			<p class="text-sm text-center text-gray-400">
				Preencha os dados para finalizar e aproveitar seu plano de internet.
			</p>
		</div>

		<!-- Indicador de progresso -->
		<div class="flex items-center justify-center w-full gap-2">
			{#each ['Dados Pessoais', 'Endereço', 'Plano'] as step, index}
				{@const stepNum = index + 1}
				<div class="flex items-center gap-2">
					<!-- Número/Ícone e Label do Step -->
					<div
						class="flex items-center transition-colors duration-200 {currentStep >= stepNum
							? 'text-orange-500'
							: 'text-gray-400'}
							{currentStep === stepNum ? 'font-semibold' : ''}"
					>
						{#if currentStep > stepNum}
							<CheckCircle2 class="h-4 w-4 md:h-5 md:w-5 text-orange-500 transition-all" />
						{:else}
							<span
								class="hidden md:flex h-6 w-6 items-center justify-center rounded-full border border-current"
							>
								{stepNum}
							</span>
						{/if}

						<span class="ml-2 text-base">
							{step}
						</span>
					</div>

					<!-- Separador entre os steps -->
					{#if stepNum < 3}
						<Separator
							class="w-32 border-[0.05rem] transition-colors duration-200
								{currentStep === stepNum + 1 ? 'bg-orange-300' : 'bg-gray-200'}"
						/>
					{/if}
				</div>
			{/each}
		</div>

		<!-- Conteúdo do formulário -->
		<div class="min-h-96 w-full">
			{#if currentStep === 1}
				<StepOne bind:formData />
			{:else if currentStep === 2}
				<StepTwo bind:formData />
			{:else}
				<StepThree bind:formData />
			{/if}
		</div>

		<!-- Botões de navegação -->
		<div class="flex justify-between w-full">
			<Button
				type="button"
				variant="secondary"
				class="w-28"
				onclick={() => navigateStep('prev')}
				disabled={currentStep === 1 || isSubmitting}
			>
				Voltar
			</Button>

			{#if currentStep < 3}
				<Button
					type="button"
					variant="default"
					class="w-28 bg-orange-600 text-white font-semibold hover:bg-orange-700"
					onclick={() => navigateStep('next')}
					disabled={isSubmitting}
				>
					Continuar
				</Button>
			{:else}
				<Button
					type="submit"
					variant="default"
					class="w-28 bg-orange-600 text-white font-semibold hover:bg-orange-700"
					disabled={isSubmitting}
				>
					{isSubmitting ? 'Enviando...' : 'Finalizar'}
				</Button>
			{/if}
		</div>
	</form>
</div>
