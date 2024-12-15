<template>
    <ModalLayout>
        <div class="w-full bg-white/30 absolute h-full cursor-progress backdrop-blur-sm z-50 rounded-xl top-0 left-0 fadeIn"
                v-if="loading"></div>
        <div>
            <ModalHeader :icon="MoveSVG" :title="t('Copy files into cloud')"></ModalHeader>
            <div class="vuefinder__move-modal__content">
                <p class="vuefinder__move-modal__description">{{ t('Are you sure you want to copy these files?') }}</p>
                <div class="vuefinder__move-modal__files vf-scrollbar">
                    <div v-for="node in items" class="vuefinder__move-modal__file">
                        <div>
                            <svg class="vuefinder__move-modal__icon" xmlns="http://www.w3.org/2000/svg" fill="none"
                                viewBox="0 0 24 24" stroke="currentColor" stroke-width="1">
                                <path stroke-linecap="round" stroke-linejoin="round"
                                    d="M7 21h10a2 2 0 002-2V9.414a1 1 0 00-.293-.707l-5.414-5.414A1 1 0 0012.586 3H7a2 2 0 00-2 2v14a2 2 0 002 2z" />
                            </svg>
                        </div>
                        <div class="vuefinder__move-modal__file-name">{{ node.path }}</div>
                    </div>
                </div>
                <message v-if="message.length" @hidden="message = ''" error>{{ message }}</message>
            </div>
        </div>
        <template v-slot:buttons>
            <button type="button" @click="copy" class="vf-btn vf-btn-primary">{{ t('Copy') }}</button>
            <button type="button" @click="app.modal.close()" class="vf-btn vf-btn-secondary">{{ t('Cancel') }}</button>

            <div class="vuefinder__move-modal__selected-items">{{ t('%s item(s) selected.', items.length) }}</div>

        </template>
    </ModalLayout>
</template>

<script setup>
import ModalLayout from './ModalLayout.vue';
import { inject, ref } from 'vue';
import Message from '../Message.vue';

import ModalHeader from "./ModalHeader.vue";
import MoveSVG from "../icons/move.svg";

const app = inject('ServiceContainer');
const { t } = app.i18n;
const message = ref('');
const loading = ref(false)

const items = ref(app.modal.data.items);

const copy = () => {
    loading.value = true;
    app.emitter.emit('vf-fetch', {
        params: {
            q: 'copyToCloud',
            m: 'post',
            adapter: app.fs.adapter,
            path: app.fs.data.dirname
        },
        body: {
            items: items.value.map(({ path, type }) => ({ path, type })),
        },
        onSuccess: () => {
            loading.value = false;
            app.emitter.emit('vf-toast-push', { label: t('Files Copied Successfully.') });
        },
        onError: (e) => {
            loading.value = false;
            message.value = t(e.message);
        }
    })
}
</script>
