<template>
    <div class="columns is-centered">
        <div class="column is-three-quarters-desktop is-full-touch">
            <enso-form class="box form-box has-background-light raises-on-hover"/>
        </div>
    </div>
</template>
<router>
{
    name: 'authors.create'
}
</router>
<script>
import { EnsoForm } from '@enso-ui/forms/bulma';
import { ref, computed, useStore } from 'vue';

export default {
layout: 'auth',
    meta: {
        permission: { name: 'authors menu' },
        title: 'Authors - Create'
    },

    components: { EnsoForm },
};
</script>

<style lang="scss">
</style>
