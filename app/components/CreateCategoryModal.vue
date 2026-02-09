<script setup lang="ts">
    import type { FormError } from '@nuxt/ui';
import type { CreateCategoryRequest } from '~/lib/openapi';

    const props = defineProps<{
      accountId: string,
      visible: boolean
    }>();

    const emit = defineEmits<{
      closed: [],
      submitted: [req: CreateCategoryRequest]
    }>();

    const newCategoryState = reactive({
        name: '',
        balance: 0,
        refillValue: 0
    });

    type Schema = typeof newCategoryState;

    function validate(state: Partial<Schema>): FormError[] {
        const errors = [];
        if(!state.name) errors.push({ name: 'name', message: 'Required' });
        if(!state.balance) errors.push({ name: 'balance', message: 'Required' });
        if(!state.refillValue) errors.push({ name: 'refillValue', message: 'Required' });
        return errors;
    }
</script>

<template>
    <UModal :open="props.visible">
      <template #content>
        <UCard>
          <template #header>
            <h3 class="text-lg font-semibold">Create New Category</h3>
          </template>
          
          <UForm
            class="space-y-4"
            :state="newCategoryState"
            :validate="validate"
          >
            <UFormField label="Name">
              <UInput v-model="newCategoryState.name" />
            </UFormField>
            
            <UFormField label="Balance">
              <UInput v-model="newCategoryState.balance" type="number" />
            </UFormField>
            
            <UFormField label="Refill Value">
              <UInput v-model="newCategoryState.refillValue" type="number" />
            </UFormField>
          </UForm>
          
          <template #footer>
            <div class="flex justify-end space-x-2">
              <UButton 
                color="neutral" 
                variant="ghost"
                @click="emit('closed')"
              >
                Cancel
              </UButton>
              <UButton 
                color="primary" 
                variant="solid"
                @click="emit('submitted', {
                  accountId: props.accountId,
                  name: newCategoryState.name, 
                  balance: newCategoryState.balance, 
                  refillValue: newCategoryState.refillValue
                })"
              >
                Create
              </UButton>
            </div>
          </template>
        </UCard>
      </template>
    </UModal>
</template>
