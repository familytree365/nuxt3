<template>
    <div class="columns is-centered">
        <div class="column is-three-quarters-desktop is-full-touch">
            <enso-form class="box form-box has-background-light raises-on-hover"/>
        </div>
    </div>
</template>
<router>
{
    name: 'sources.edit'
}
</router>
<script>
import { EnsoForm } from '@enso-ui/forms/bulma';
import { ref, computed, useStore } from 'vue';
export default {
    name: 'Edit',
    layout: 'auth',
    meta: {
        permission: { name: 'sources menu' },
        title: 'Sources - Edit'
    },
    components: { EnsoForm },
};
</script>

<style lang="scss">
</style>
