<script setup lang="ts">
import type { FormError } from '@nuxt/ui';
import { ref } from 'vue';

const props = defineProps<{
    visible: boolean,
    categoryId: string,
    categoryName: string
}>();

const emit = defineEmits<{
    closed: [],
    submitted: [amount: number, isAddition: boolean]
}>();

const transactionAmount = ref(0);
const isAddition = ref(true);

type Schema = typeof transactionAmount;

function validate(state: Partial<Schema>): FormError[] {
    const errors = [];
    if (!state) errors.push({ name: 'amount', message: 'Required' });
    return errors;
}

function submitRequest() {
    emit('submitted', transactionAmount.value, isAddition.value);
    transactionAmount.value = 0;
    isAddition.value = true;
}
</script>

<template>
    <UModal :open="props.visible">
        <template #content>
            <UCard>
                <template #header>
                    <h3 class="text-lg font-semibold">Create Transaction for {{ props.categoryName }}</h3>
                </template>

                <UForm class="space-y-4" :state="transactionAmount" :validate="validate">
                    <UFormField label="Amount">
                        <NumberInput v-model="transactionAmount" />
                    </UFormField>

                    <div class="flex space-x-4 justify-center">
                        <UButton color="success" variant="solid" @click="isAddition = true; submitRequest()"
                            :disabled="!transactionAmount">
                            Add Money (+)
                        </UButton>
                        <UButton color="error" variant="solid" @click="isAddition = false; submitRequest()"
                            :disabled="!transactionAmount">
                            Subtract Money (-)
                        </UButton>
                    </div>
                </UForm>

                <template #footer>
                    <div class="flex justify-end space-x-2">
                        <UButton color="neutral" variant="ghost" @click="emit('closed')">
                            Cancel
                        </UButton>
                    </div>
                </template>
            </UCard>
        </template>
    </UModal>
</template>