<template>
  <div class="polaris-card">
    <div class="polaris-card__section">
      <div class="polaris-block-stack">
        <!-- Method & URL -->
        <div class="polaris-inline polaris-inline--gap-tight polaris-inline--align-end">
          <div class="polaris-text-field" style="width: 100px;">
            <label class="polaris-text-field__label polaris-text-field__label--required">Method</label>
            <select
              class="polaris-select__input"
              :value="config?.method || 'GET'"
              @change="updateField('method', $event.target.value)"
            >
              <option value="GET">GET</option>
              <option value="POST">POST</option>
              <option value="PUT">PUT</option>
              <option value="PATCH">PATCH</option>
              <option value="DELETE">DELETE</option>
            </select>
          </div>

          <div class="polaris-text-field polaris-text-field--flex">
            <label class="polaris-text-field__label polaris-text-field__label--required">URL</label>
            <input
              type="text"
              class="polaris-text-field__input"
              :value="config?.url || ''"
              @input="updateField('url', $event.target.value)"
              placeholder="https://api.example.com/endpoint"
            />
          </div>
        </div>

        <!-- Headers -->
        <div class="polaris-text-field">
          <label class="polaris-text-field__label">Headers</label>
          <div class="polaris-block-stack polaris-block-stack--tight">
            <div
              v-for="(header, idx) in headersArray"
              :key="idx"
              class="polaris-inline polaris-inline--gap-tight"
            >
              <input
                type="text"
                class="polaris-text-field__input"
                :value="header.key"
                @input="updateHeader(idx, 'key', $event.target.value)"
                placeholder="Header name"
              />
              <input
                type="text"
                class="polaris-text-field__input"
                :value="header.value"
                @input="updateHeader(idx, 'value', $event.target.value)"
                placeholder="Header value"
              />
              <button
                class="polaris-button polaris-button--plain polaris-button--icon-only"
                @click="removeHeader(idx)"
                title="Remove header"
              >
                <svg viewBox="0 0 20 20" class="polaris-icon polaris-icon--small">
                  <path d="M11.414 10l4.293-4.293a1 1 0 00-1.414-1.414L10 8.586 5.707 4.293a1 1 0 00-1.414 1.414L8.586 10l-4.293 4.293a1 1 0 101.414 1.414L10 11.414l4.293 4.293a1 1 0 001.414-1.414L11.414 10z" fill="currentColor"/>
                </svg>
              </button>
            </div>
            <button class="polaris-button polaris-button--plain" @click="addHeader">
              <svg viewBox="0 0 20 20" class="polaris-icon polaris-icon--small">
                <path d="M10 5a1 1 0 011 1v3h3a1 1 0 110 2h-3v3a1 1 0 11-2 0v-3H6a1 1 0 110-2h3V6a1 1 0 011-1z" fill="currentColor"/>
              </svg>
              Add header
            </button>
          </div>
        </div>

        <!-- Body -->
        <div v-if="config?.method !== 'GET'" class="polaris-text-field">
          <label class="polaris-text-field__label">Body (JSON)</label>
          <textarea
            class="polaris-text-field__input polaris-text-field__input--multiline polaris-text-field__input--monospace"
            :value="bodyString"
            @input="handleBodyChange($event.target.value)"
            placeholder='{"key": "value"}'
            rows="6"
          ></textarea>
          <div v-if="bodyError" class="polaris-text-field__error">
            <svg viewBox="0 0 20 20" class="polaris-icon polaris-icon--small">
              <path d="M10 18a8 8 0 110-16 8 8 0 010 16zM9 9a1 1 0 002 0V7a1 1 0 00-2 0v2zm0 4a1 1 0 102 0 1 1 0 00-2 0z" fill="currentColor"/>
            </svg>
            {{ bodyError }}
          </div>
        </div>

        <!-- Timeout & Retry -->
        <div class="polaris-inline polaris-inline--gap-tight">
          <div class="polaris-text-field polaris-text-field--flex">
            <label class="polaris-text-field__label">Timeout (seconds)</label>
            <input
              type="number"
              class="polaris-text-field__input"
              :value="config?.timeout_seconds || 30"
              @input="updateField('timeout_seconds', parseInt($event.target.value) || 30)"
              min="1"
              max="300"
            />
          </div>

          <div class="polaris-text-field polaris-text-field--flex">
            <label class="polaris-text-field__label">Retry count</label>
            <input
              type="number"
              class="polaris-text-field__input"
              :value="config?.retry_count || 0"
              @input="updateField('retry_count', parseInt($event.target.value) || 0)"
              min="0"
              max="10"
            />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { computed, ref } from 'vue';

export default {
  name: 'ApiConfig',
  props: {
    config: { type: Object, required: true },
  },
  emits: ['update'],
  setup(props, { emit }) {
    const bodyError = ref('');

    const bodyString = computed(() => {
      const body = props.config?.body;
      if (!body) return '';
      if (typeof body === 'object') {
        return JSON.stringify(body, null, 2);
      }
      return body;
    });

    const headersArray = computed(() => {
      const headers = props.config?.headers || {};
      const entries = Object.entries(headers);
      if (entries.length === 0) {
        return [{ key: '', value: '' }];
      }
      return entries.map(([key, value]) => ({ key, value }));
    });

    const updateField = (field, value) => {
      emit('update', { ...props.config, [field]: value });
    };

    const handleBodyChange = (value) => {
      if (!value) {
        emit('update', { ...props.config, body: null });
        bodyError.value = '';
        return;
      }
      try {
        const parsed = JSON.parse(value);
        emit('update', { ...props.config, body: parsed });
        bodyError.value = '';
      } catch (e) {
        emit('update', { ...props.config, body: value });
        bodyError.value = 'Invalid JSON: ' + e.message;
      }
    };

    const updateHeader = (index, field, value) => {
      const headers = { ...(props.config?.headers || {}) };
      const entries = Object.entries(headers);
      
      if (entries[index]) {
        const oldKey = entries[index][0];
        if (field === 'key') {
          const oldValue = headers[oldKey];
          delete headers[oldKey];
          if (value) {
            headers[value] = oldValue;
          }
        } else {
          headers[oldKey] = value;
        }
      } else if (field === 'key' && value) {
        headers[value] = '';
      } else if (field === 'value' && entries.length === 0) {
        headers[''] = value;
      }
      
      emit('update', { ...props.config, headers });
    };

    const addHeader = () => {
      const headers = { ...(props.config?.headers || {}) };
      let newKey = '';
      let counter = 0;
      while (headers.hasOwnProperty(newKey)) {
        counter++;
        newKey = `header_${counter}`;
      }
      headers[newKey] = '';
      emit('update', { ...props.config, headers });
    };

    const removeHeader = (index) => {
      const headers = { ...(props.config?.headers || {}) };
      const entries = Object.entries(headers);
      if (entries[index]) {
        delete headers[entries[index][0]];
      }
      emit('update', { ...props.config, headers });
    };

    return {
      bodyError,
      bodyString,
      headersArray,
      updateField,
      handleBodyChange,
      updateHeader,
      addHeader,
      removeHeader,
    };
  },
};
</script>

<style lang="scss" scoped>
@import '../styles/polaris-weweb-styles.scss';

.polaris-card {
  @include polaris-tokens;
  @include polaris-card;
}

.polaris-card__section { @include polaris-card-section; }

.polaris-block-stack {
  @include polaris-block-stack;
  gap: var(--p-space-300);
  &--tight { gap: var(--p-space-200); }
}

.polaris-inline {
  @include polaris-inline;
  &--gap-tight { gap: var(--p-space-200); }
  &--align-end { align-items: flex-end; }
}

.polaris-text-field {
  display: flex;
  flex-direction: column;
  gap: var(--p-space-100);
  &--flex { flex: 1; }
}

.polaris-text-field__label {
  @include polaris-label;
  &--required::after { content: ' *'; color: var(--p-color-text-critical); }
}

.polaris-text-field__input {
  @include polaris-input;
  &--multiline { @include polaris-textarea; }
  &--monospace { font-family: var(--p-font-family-mono); font-size: var(--p-font-size-300); }
}

.polaris-text-field__error { @include polaris-error-text; }
.polaris-select__input { @include polaris-select; }

.polaris-button {
  @include polaris-button-base;
  &--plain { @include polaris-button-plain; }
  &--icon-only { @include polaris-button-icon-only; }
}

.polaris-icon {
  @include polaris-icon;
  &--small { @include polaris-icon-small; }
}
</style>

