<template>
    <div class="columns is-centered">
        <div class="column is-three-quarters-desktop is-full-touch">
            <enso-form class="box form-box has-background-light raises-on-hover"/>
        </div>
    </div>
</template>
<router>
{
    name: 'chan.create'
}
</router>
<script>
import { EnsoForm } from '@enso-ui/forms/bulma';
import { ref, compured, useStore } from 'vue';

export default {
layout: 'auth',
    meta: {
        permission: { name: 'chan menu' },
        title: 'Chan - Create'
    },

    components: { EnsoForm },
};
</script>

<style lang="scss">
</style>
