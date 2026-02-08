<script setup lang="ts">
    import type { SignInModel } from '~/lib/openapi';
    const signInState = useState<SignInModel | undefined>('signInState', () => undefined);
</script>

<template>
    <p>Signed in as {{ signInState?.userModel }} ({{ signInState?.accessToken }})</p>
</template>