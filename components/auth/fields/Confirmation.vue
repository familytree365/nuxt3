<template>
    <div class="field">
        <div class="control has-icons-left has-icons-right">
            <input :value="value"
                class="input"
                :type="meta.content"
                :class="{ 'is-danger': errors.has('password'), 'is-success': successful }"
                :placeholder="i18n('Repeat Password')"
                @input="$emit('input', $event.target.value); errors.clear('password')">
            <span class="icon is-small is-left">
                <fa icon="lock"/>
            </span>
            <reveal-password :meta="meta"
                :class="{ 'mr-5': match || successful || errors.has('password')}"
                v-if="value && !successful"/>
            <span v-if="errors.has('password')"
                class="icon is-small is-right has-text-danger">
                <fa icon="exclamation-triangle"/>
            </span>
            <span v-if="match && !errors.has('password') || successful"
                class="icon is-small is-right has-text-success">
                <fa icon="check"/>
            </span>
        </div>
        <p class="has-text-danger is-size-7"
            v-if="errors.has('password')">
            {{ errors.get('password') }}
        </p>
    </div>
</template>

<script>
import { library } from '@fortawesome/fontawesome-svg-core';
import { faCheck, faExclamationTriangle, faLock } from '@fortawesome/free-solid-svg-icons';
import { focus } from '@enso-ui/directives';
import RevealPassword from '@enso-ui/forms/src/bulma/parts/RevealPassword.vue';
import { ref, computed, useState } from 'vue';

library.add(faCheck, faExclamationTriangle, faLock);

export default {
    name: 'Confirmation',

    directives: { focus },

    inject: ['errors', 'i18n', 'state'],

    components: { RevealPassword },

    props: {
        match: {
            type: Boolean,
            required: true,
        },
        value: {
            type: String,
            required: true,
        },
    },
    setup() {
        const successful = computed(() => {
            return this.state.successful;
        })
        const meta = ref({
            content: 'password'
        })
    }
};
</script>
