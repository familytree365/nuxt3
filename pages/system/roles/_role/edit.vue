<template>
    <div class="columns is-centered">
        <div class="column is-three-quarters-desktop is-full-touch">
            <enso-form class="box has-background-light raises-on-hover"
                @ready="ready = true"
                ref="form">
                <template v-slot:actions-left
                    v-if="ready">
                    <div class="level-item">
                        <a class="button is-warning"
                            @click="$router.push({
                                name: 'system.roles.configure',
                                params: { role: $refs.form.routeParam('role') }
                            })">
                            <span class="is-hidden-mobile">
                                {{ i18n('Configure') }}
                            </span>
                            <span class="icon">
                                <fa icon="sliders-h"/>
                            </span>
                            <span class="is-hidden-mobile"/>
                        </a>
                    </div>
                    <div class="level-item">
                        <a class="button is-link"
                            @click="writeConfig">
                            <span class="is-hidden-mobile">
                                {{ i18n('File') }}
                            </span>
                            <span class="icon">
                                <fa icon="save"/>
                            </span>
                            <span class="is-hidden-mobile"/>
                        </a>
                    </div>
                </template>
            </enso-form>
        </div>
    </div>
</template>
<router>
{
    name: 'system.roles.edit',
}
</router>

<script>
import { library } from '@fortawesome/fontawesome-svg-core';
import { faSave, faSlidersH } from '@fortawesome/free-solid-svg-icons';
import { EnsoForm } from '@enso-ui/forms/bulma';
import { ref, computed, useStore } from 'vue';

library.add([faSave, faSlidersH]);

export default {
    meta:{ breadcrumb: 'edit', title: 'Edit Role' },
    inject: ['errorHandler', 'i18n', 'route', 'routerErrorHandler', 'toastr'],
    components: { EnsoForm },
    setup() {
        const ready = ref(false)
        function writeConfig() {
            this.$axios.post(this.route(
                'system.roles.permissions.write',
                this.$refs.form.routeParam('role'),
            )).then(({ data }) => this.toastr.success(data.message))
                .catch(this.errorHandler);
        }
    }
}
</script>
