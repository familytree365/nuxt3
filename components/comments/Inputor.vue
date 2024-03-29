<template>
    <div class="animated fadeIn atwho-wrapper"
        @keyup="filter"
        @keydown.up="onUp"
        @keydown.down="onDown"
        @keydown.enter="onEnter">
        <div v-show="items.length"
            v-click-outside="hide"
            class="atwho dropdown-menu">
            <div class="dropdown-content">
                <a v-for="(item, index) in items"
                    :key="index"
                    :class="['dropdown-item', { 'is-active': index === position}]"
                    @mousemove="position = index"
                    @click="selectItem">
                    <article class="media">
                        <div class="media-left">
                            <figure class="image is-24x24">
                                <img class="is-rounded"
                                    :src="'/api/core/avatars/' + item.avatar.id">
                            </figure>
                        </div>
                        <div class="media-content">
                            <div class="content"
                                v-html="highlight(item.person.name)"/>
                        </div>
                    </article>
                </a>
            </div>
        </div>
        <div class="field">
            <p class="control">
                <textarea v-model="comment.body"
                    v-focus
                    class="textarea vue-comment"
                    :placeholder="i18n('Type a new comment')"
                    @keyup.shift.enter="$emit('save')"/>
            </p>
        </div>
    </div>
</template>

<script>
import { mapState } from 'vuex';
import debounce from 'lodash/debounce';
import getCaretCoordinates from 'textarea-caret';
import { focus, clickOutside } from '@enso-ui/directives';
import { ref, computed, useStore, watch } from 'vue';

export default {
    name: 'Inputor',

    directives: { focus, clickOutside },

    inject: ['errorHandler', 'i18n', 'route'],

    props: {
        comment: {
            type: Object,
            required: true,
        },
    },
    setup() {
        const items = ref([])
        const position = ref(null)
        const query = ref(null)
        const store = useStore()
        return {
            one: computed(() => store.state[user])
        }
        const hasText = computed(() => {
            return this.comment.body.trim();
        })
        const atwhoContainer = computed(() => {
            return this.$el.querySelector('.atwho');
        })
        const textarea = computed(() => {
            return this.$el.querySelector('textarea');
        })
        const query = ref('')
        watch(query, () => {
            if (this.query !== null) {
                this.fetch();
            }
        })
        created(() => {
            this.fetch = debounce(this.fetch, 200);
        })
        function fetch() {
            this.$axios.get(this.route('core.comments.users'), {
                params: { query: this.query, paginate: 6 },
            }).then(({ data }) => {
                this.items = data
                    .filter(({ id }) => id !== this.user.id);

                if (this.items.length) {
                    this.position = 0;
                }
            }).catch(this.errorHandler);
        }
        function filter(event) {
            const arg = this.comment.body
                .substr(0, this.textarea.selectionEnd)
                .split(' ')
                .pop();

            const showAtwho = arg[0] === '@'
                && (this.comment.body.length === this.textarea.selectionEnd
                    || this.comment.body[this.textarea.selectionEnd] === ' ');

            if (!showAtwho) {
                this.hide();
            }

            if (showAtwho) {
                this.query = arg.substring(1);
                this.atwho(event);
            }
        }
        function atwho(event) {
            const caret = getCaretCoordinates(event.target, event.target.selectionEnd);
            this.atwhoContainer.style.top = `${caret.top + 25}px`;
            this.atwhoContainer.style.left = `${caret.left - 15}px`;
        }
        function onUp(event) {
            if (this.position > 0) {
                this.position--;
                event.preventDefault();
            }
        }
        function onDown(event) {
            if (this.position < this.items.length - 1) {
                this.position++;
                event.preventDefault();
            }
        }
        function onEnter(event) {
            if (this.position !== null) {
                this.selectItem();
                event.preventDefault();
            }
        }
        function selectItem() {
            const { search, replace } = this.transform();

            this.comment.body = this.comment.body.replace(search, replace);

            this.tagUser();
            this.hide();

            this.$nextTick(() => {
                this.textarea.selectionEnd = replace.length;
            });
        }
        function hide() {
            this.items = [];
            this.position = null;
            this.query = null;
        }
        function transform() {
            const item = this.items[this.position];
            const cursorPosition = this.textarea.selectionEnd;
            let search = this.comment.body.substr(0, cursorPosition);
            const atPosition = search.lastIndexOf('@');
            if (this.comment.body[search.length] === ' ') {
                search += ' ';
            }
            let replace = search.substr(0, atPosition + 1) + item.person.name;
            replace += ' ';
            return { search, replace };
        }
        function highlight(item) {
            return item
                .replace(new RegExp(`(${this.query})`, 'gi'), '<b>$1</b>');
        }
        function tagUser() {
            const user = this.items[this.position];
            this.comment.taggedUsers.push({
                id: user.id,
                name: user.person.name,
            });
        }
    }
};
</script>

<style lang="scss">
    .atwho-wrapper {
        position: relative;

        .dropdown-menu {
            display: unset;
            top: unset;
            left: unset;
            min-width: unset;
            padding-top: unset;

            .dropdown-content {
                width: fit-content;
                padding: 0.25rem 0;

                a.dropdown-item {
                    padding: .1rem 0.5rem;
                }

                .media {
                    border-top: none;
                    padding-top: 0;

                    .media-left {
                        margin-right: 0.5rem;
                    }
                }
            }
        }

        textarea.vue-comment {
            resize: none;
            word-wrap: break-word;

            &:not([rows]) {
                min-height: 90px;
            }
        }
    }
</style>
