<script setup lang="ts">
    import type { FormError } from '@nuxt/ui';
    import { AccountApi } from '~/lib/openapi';
    import { deleteAccount, refreshAccountState, useAccountState, useDefaultAccountName } from '~/lib/services/account-service';
    import { getApiConfiguration } from '~/lib/services/auth-state';

    const defaultAccountIdStorage = useDefaultAccountName();
    const accounts = useAccountState();
    const creatingAccount = ref(false);

    async function onDeleteAccount(event: MouseEvent, id: string) {
        event.preventDefault();
        await deleteAccount(id);
    }

    const createAccountState = reactive({
        name: undefined,
    });

    type Schema = typeof createAccountState;

    function validateCreateAccount(state: Partial<Schema>): FormError[] {
        const errors = [];
        if(!state.name) errors.push({ name: 'name', message: 'Required' });
        return errors;
    }

    async function onCreateAccount() {
        const api = new AccountApi(getApiConfiguration(true));
        try { await api.accountCreatePost({ name: createAccountState.name }); }
        catch { return; }
        creatingAccount.value = false;
        await refreshAccountState();
    }
</script>

<template>
    <div>
        <UPageList>
            <UPageCard 
                v-for="account in accounts"
                :key="account.id"
                variant="subtle"
                to="/"
                @click="defaultAccountIdStorage=account.name"
            >
                <template #body>
                    <UUser :name="account.name" size="xl" />
                </template>
                <template #footer>
                    <UButton color="error" @click="onDeleteAccount($event, account.id)">Delete</UButton>
                </template>
            </UPageCard>
        </UPageList>
        <UButton class="mt-4" @click="creatingAccount=true">Create Account</UButton>
        <UModal :open="creatingAccount">
            <template #content>
                <UForm 
                    class="space-y-4"
                    :validate="validateCreateAccount" 
                    :state="createAccountState" 
                    @submit="onCreateAccount"
                >
                    <UFormField label="Name">
                        <UInput v-model="createAccountState.name" type="text"  />
                    </UFormField>
                    <UButton type="submit">Submit</UButton>
                </UForm>
            </template>
        </UModal>
    </div>
</template>