<script setup lang="ts">
  import { CategoryApi, Role, type CategoryModel, type CreateCategoryRequest } from '~/lib/openapi';
  import { useAccountState, useDefaultAccountName } from '~/lib/services/account-service';
  import { getApiConfiguration, useSignInState } from '~/lib/services/auth-state';
  import { ref, watch } from 'vue';

  const signInState = useSignInState();
  const accountState = useAccountState();
  const account = computed(() => accountState.value?.find(a => a.name === defaultAccountName.value));

  const defaultAccountName = useDefaultAccountName();

  const categories = ref<CategoryModel[]>([]);
  const loading = ref(true);
  const showCreateCategoryModal = ref(false);

  async function loadCategories() {
    if (!accountState.value || !defaultAccountName.value) {
      loading.value = false;
      return;
    }

    
    if (!account.value) {
      loading.value = false;
      return;
    }

    try {
      const api = new CategoryApi(getApiConfiguration(true));
      categories.value = await api.categoryForAccountAccountIdGet({ accountId: account.value.id });
    } catch (error) {
      console.error('Failed to load categories:', error);
    } finally {
      loading.value = false;
    }
  }

  // Load categories when default account changes
  watch(defaultAccountName, () => {
    if (defaultAccountName.value) {
      loadCategories();
    }
  });

  onMounted(() => {
    if (!signInState.value) {
      navigateTo('/login');
    }

    // Redirect to accounts page if no default account is selected
    if (!defaultAccountName.value) {
      navigateTo('/accounts');
    }

    loadCategories();
  });

  async function createCategory(state: CreateCategoryRequest) {
    if (!accountState.value || !defaultAccountName.value) {
      return;
    }

    const account = accountState.value.find(a => a.name === defaultAccountName.value);
    
    if (!account) {
      return;
    }

    try {
      const api = new CategoryApi(getApiConfiguration(true));
      const request: CreateCategoryRequest = {
        accountId: account.id,
        name: state.name,
        balance: state.balance,
        refillValue: state.refillValue
      };
      
      await api.categoryCreatePost({ createCategoryRequest: request });

      showCreateCategoryModal.value = false;
      state.name = '';
      state.balance = 0;
      state.refillValue = 0;

      loadCategories();
    } catch (error) {
      console.error('Failed to create category:', error);
    }
  }

  async function deleteCategory(categoryId: string) {
    if (!confirm('Are you sure you want to delete this category?')) {
      return;
    }

    try {
      const api = new CategoryApi(getApiConfiguration(true));
      await api.categoryDeleteIdDelete({ id: categoryId });
      loadCategories();
    } catch (error) {
      console.error('Failed to delete category:', error);
      alert('Failed to delete category');
    }
  }
</script>

<template>
  <div>
    <h1 class="text-2xl font-bold mb-4">Categories</h1>
    
    <UButton 
      v-if="signInState?.userModel.role == Role.Admin"
      color="primary" 
      variant="solid"
      icon="i-heroicons-plus-circle"
      class="mb-4"
      @click="showCreateCategoryModal = true"
    >
      New Category
    </UButton>
    
    <div v-if="loading" class="flex justify-center items-center h-64">
      <ULoading />
    </div>
    
    <div v-else-if="categories.length === 0" class="text-center py-8">
      <p>No categories found. Create some categories in the accounts page.</p>
    </div>
    
    <div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      <UCard 
        v-for="category in categories" 
        :key="category.id"
        class="relative"
      >
        <template #header>
          <div class="flex justify-between items-center">
            <h3 class="text-lg font-semibold">{{ category.name }}</h3>
          </div>
        </template>
        
        <p>Balance: {{ category.balance }}</p>
        
        <template #footer>
          <div class="flex justify-end space-x-2">
            <UButton 
              color="success" 
              variant="ghost"
              icon="i-heroicons-plus-circle"
              size="xl"
            />
            <UButton 
              color="error" 
              variant="ghost"
              icon="i-heroicons-minus-circle"
              size="xl"
            />
            <UButton 
              v-if="signInState?.userModel.role == Role.Admin"
              color="error" 
              variant="ghost"
              icon="i-heroicons-trash"
              size="xl"
              @click="deleteCategory(category.id)"
            />
          </div>
        </template>
      </UCard>
    </div>
    
    <CreateCategoryModal 
      :account-id="account?.id ?? ''" 
      :visible="showCreateCategoryModal"
      @submitted="createCategory($event)"
      @closed="showCreateCategoryModal=false"
    />
    
  </div>
</template>
