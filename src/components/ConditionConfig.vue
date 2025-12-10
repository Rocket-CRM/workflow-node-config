<template>
  <div class="polaris-card">
    <!-- Groups Operator -->
    <div v-if="(config?.groups?.length || 0) > 1" class="polaris-card__section">
      <div class="polaris-inline polaris-inline--space-between">
        <span class="polaris-text polaris-text--body-md">Combine groups with</span>
        <div class="polaris-button-group polaris-button-group--segmented">
          <button
            :class="['polaris-button polaris-button--segmented', { 'polaris-button--segmented-selected': config?.groups_operator === 'AND' }]"
            @click="$emit('update', { ...config, groups_operator: 'AND' })"
          >AND</button>
          <button
            :class="['polaris-button polaris-button--segmented', { 'polaris-button--segmented-selected': config?.groups_operator === 'OR' }]"
            @click="$emit('update', { ...config, groups_operator: 'OR' })"
          >OR</button>
        </div>
      </div>
    </div>

    <!-- Groups -->
    <div class="polaris-card__section">
      <div class="polaris-block-stack">
        <div
          v-for="(group, gIdx) in (config?.groups || [])"
          :key="group?.id || gIdx"
          class="polaris-card polaris-card--subdued"
        >
          <div class="polaris-card__section">
            <div class="polaris-inline polaris-inline--space-between">
              <span class="polaris-text polaris-text--heading-sm">Group {{ gIdx + 1 }}</span>
              <button
                class="polaris-button polaris-button--plain polaris-button--critical polaris-button--icon-only"
                @click="removeGroup(group?.id)"
                :disabled="(config?.groups?.length || 0) <= 1"
                title="Remove group"
              >
                <svg viewBox="0 0 20 20" class="polaris-icon polaris-icon--small">
                  <path d="M14 4h-4V3A1 1 0 009 2H7a1 1 0 00-1 1v1H2a1 1 0 000 2h1v10a2 2 0 002 2h6a2 2 0 002-2V6h1a1 1 0 100-2zM8 4V3h.5v1H8zm3 12H5V6h6v10z" fill="currentColor"/>
                </svg>
              </button>
            </div>

            <!-- Collection Selection -->
            <div class="polaris-text-field">
              <label class="polaris-text-field__label">Collection</label>
              <select
                class="polaris-select__input"
                :value="group?.collection || ''"
                @change="updateGroupCollection(group?.id, $event.target.value)"
              >
                <option value="">Select collection...</option>
                <option
                  v-for="col in collections"
                  :key="col?.name"
                  :value="col?.name"
                >{{ col?.label || col?.name }}</option>
              </select>
            </div>

            <!-- Conditions -->
            <div class="polaris-condition-list">
              <div
                v-for="(condition, cIdx) in (group?.conditions || [])"
                :key="condition?.id || cIdx"
                class="polaris-condition-item"
              >
                <!-- AND/OR badge between conditions -->
                <div v-if="cIdx > 0" class="polaris-condition-operator">
                  <div class="polaris-button-group polaris-button-group--segmented polaris-button-group--small">
                    <button
                      :class="['polaris-button polaris-button--segmented polaris-button--slim', { 'polaris-button--segmented-selected': group?.operator === 'AND' }]"
                      @click="updateGroupOperator(group?.id, 'AND')"
                    >AND</button>
                    <button
                      :class="['polaris-button polaris-button--segmented polaris-button--slim', { 'polaris-button--segmented-selected': group?.operator === 'OR' }]"
                      @click="updateGroupOperator(group?.id, 'OR')"
                    >OR</button>
                  </div>
                </div>

                <div class="polaris-condition-fields">
                  <!-- Field Selection -->
                  <div class="polaris-text-field polaris-text-field--flex">
                    <select
                      class="polaris-select__input"
                      :value="condition?.field || ''"
                      @change="updateConditionField(group?.id, condition?.id, $event.target.value)"
                      :disabled="!group?.collection"
                    >
                      <option value="">Field...</option>
                      <option
                        v-for="field in getFieldsForCollection(group?.collection)"
                        :key="field?.name"
                        :value="field?.name"
                      >{{ field?.label || field?.name }}</option>
                    </select>
                  </div>

                  <!-- Operator Selection -->
                  <div class="polaris-text-field polaris-text-field--operator">
                    <select
                      class="polaris-select__input"
                      :value="condition?.operator || 'equals'"
                      @change="updateConditionOperator(group?.id, condition?.id, $event.target.value)"
                      :disabled="!condition?.field"
                    >
                      <option
                        v-for="op in getOperatorsForField(group?.collection, condition?.field)"
                        :key="op.value"
                        :value="op.value"
                      >{{ op.label }}</option>
                    </select>
                  </div>

                  <!-- Value Input -->
                  <div v-if="isValueRequired(condition?.operator)" class="polaris-text-field polaris-text-field--flex">
                    <input
                      type="text"
                      class="polaris-text-field__input"
                      :value="condition?.value || ''"
                      @input="updateConditionValue(group?.id, condition?.id, $event.target.value)"
                      placeholder="Value"
                    />
                  </div>

                  <!-- Remove Condition -->
                  <button
                    class="polaris-button polaris-button--plain polaris-button--icon-only"
                    @click="removeCondition(group?.id, condition?.id)"
                    :disabled="(group?.conditions?.length || 0) <= 1"
                    title="Remove condition"
                  >
                    <svg viewBox="0 0 20 20" class="polaris-icon polaris-icon--small">
                      <path d="M11.414 10l4.293-4.293a1 1 0 00-1.414-1.414L10 8.586 5.707 4.293a1 1 0 00-1.414 1.414L8.586 10l-4.293 4.293a1 1 0 101.414 1.414L10 11.414l4.293 4.293a1 1 0 001.414-1.414L11.414 10z" fill="currentColor"/>
                    </svg>
                  </button>
                </div>
              </div>

              <!-- Add Condition Button -->
              <button
                class="polaris-button polaris-button--plain polaris-button--full-width"
                @click="addCondition(group?.id)"
                :disabled="!group?.collection"
              >
                <svg viewBox="0 0 20 20" class="polaris-icon polaris-icon--small">
                  <path d="M10 5a1 1 0 011 1v3h3a1 1 0 110 2h-3v3a1 1 0 11-2 0v-3H6a1 1 0 110-2h3V6a1 1 0 011-1z" fill="currentColor"/>
                </svg>
                Add condition
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Add Group Button -->
      <button class="polaris-button polaris-button--outline polaris-button--full-width" @click="addGroup">
        <svg viewBox="0 0 20 20" class="polaris-icon polaris-icon--small">
          <path d="M10 5a1 1 0 011 1v3h3a1 1 0 110 2h-3v3a1 1 0 11-2 0v-3H6a1 1 0 110-2h3V6a1 1 0 011-1z" fill="currentColor"/>
        </svg>
        Add group
      </button>
    </div>
  </div>
</template>

<script>
const OPERATORS_BY_TYPE = {
  string: [
    { value: 'equals', label: 'equals' },
    { value: 'not_equals', label: 'not equals' },
    { value: 'contains', label: 'contains' },
    { value: 'not_contains', label: 'not contains' },
    { value: 'starts_with', label: 'starts with' },
    { value: 'ends_with', label: 'ends with' },
    { value: 'is_empty', label: 'is empty' },
    { value: 'is_not_empty', label: 'is not empty' },
  ],
  number: [
    { value: 'equals', label: '=' },
    { value: 'not_equals', label: '≠' },
    { value: 'greater_than', label: '>' },
    { value: 'greater_or_equal', label: '≥' },
    { value: 'less_than', label: '<' },
    { value: 'less_or_equal', label: '≤' },
    { value: 'between', label: 'between' },
  ],
  date: [
    { value: 'equals', label: 'equals' },
    { value: 'before', label: 'before' },
    { value: 'after', label: 'after' },
    { value: 'between', label: 'between' },
    { value: 'within_days', label: 'within X days' },
    { value: 'within_months', label: 'within X months' },
  ],
  boolean: [
    { value: 'is_true', label: 'is true' },
    { value: 'is_false', label: 'is false' },
  ],
  array: [
    { value: 'contains', label: 'contains' },
    { value: 'not_contains', label: 'does not contain' },
    { value: 'is_empty', label: 'is empty' },
    { value: 'is_not_empty', label: 'is not empty' },
  ],
  uuid: [
    { value: 'equals', label: 'equals' },
    { value: 'not_equals', label: 'not equals' },
    { value: 'is_empty', label: 'is empty' },
    { value: 'is_not_empty', label: 'is not empty' },
  ],
};

export default {
  name: 'ConditionConfig',
  props: {
    config: { type: Object, required: true },
    collections: { type: Array, default: () => [] },
  },
  emits: ['update'],
  setup(props, { emit }) {
    const emitUpdate = (newConfig) => {
      emit('update', newConfig);
    };

    const getFieldsForCollection = (collectionName) => {
      const collection = props.collections?.find(c => c?.name === collectionName);
      return collection?.fields || [];
    };

    const getFieldType = (collectionName, fieldName) => {
      const fields = getFieldsForCollection(collectionName);
      const field = fields.find(f => f?.name === fieldName);
      return field?.type || 'string';
    };

    const getOperatorsForField = (collectionName, fieldName) => {
      const fieldType = getFieldType(collectionName, fieldName);
      return OPERATORS_BY_TYPE[fieldType] || OPERATORS_BY_TYPE.string;
    };

    const isValueRequired = (operator) => {
      return !['is_empty', 'is_not_empty', 'is_true', 'is_false'].includes(operator);
    };

    const addGroup = () => {
      const newGroup = {
        id: crypto.randomUUID(),
        operator: 'AND',
        collection: '',
        conditions: [{
          id: crypto.randomUUID(),
          field: '',
          operator: 'equals',
          value: '',
          value_type: 'static',
        }],
      };
      emitUpdate({ ...props.config, groups: [...(props.config?.groups || []), newGroup] });
    };

    const removeGroup = (groupId) => {
      const groups = (props.config?.groups || []).filter(g => g?.id !== groupId);
      emitUpdate({ ...props.config, groups });
    };

    const updateGroupCollection = (groupId, collection) => {
      const groups = (props.config?.groups || []).map(g =>
        g?.id === groupId ? { ...g, collection } : g
      );
      emitUpdate({ ...props.config, groups });
    };

    const updateGroupOperator = (groupId, operator) => {
      const groups = (props.config?.groups || []).map(g =>
        g?.id === groupId ? { ...g, operator } : g
      );
      emitUpdate({ ...props.config, groups });
    };

    const addCondition = (groupId) => {
      const groups = (props.config?.groups || []).map(g => {
        if (g?.id === groupId) {
          return {
            ...g,
            conditions: [...(g?.conditions || []), {
              id: crypto.randomUUID(),
              field: '',
              operator: 'equals',
              value: '',
              value_type: 'static',
            }],
          };
        }
        return g;
      });
      emitUpdate({ ...props.config, groups });
    };

    const removeCondition = (groupId, conditionId) => {
      const groups = (props.config?.groups || []).map(g => {
        if (g?.id === groupId) {
          return {
            ...g,
            conditions: (g?.conditions || []).filter(c => c?.id !== conditionId),
          };
        }
        return g;
      });
      emitUpdate({ ...props.config, groups });
    };

    const updateConditionField = (groupId, conditionId, field) => {
      const groups = (props.config?.groups || []).map(g => {
        if (g?.id === groupId) {
          return {
            ...g,
            conditions: (g?.conditions || []).map(c =>
              c?.id === conditionId ? { ...c, field, value: '' } : c
            ),
          };
        }
        return g;
      });
      emitUpdate({ ...props.config, groups });
    };

    const updateConditionOperator = (groupId, conditionId, operator) => {
      const groups = (props.config?.groups || []).map(g => {
        if (g?.id === groupId) {
          return {
            ...g,
            conditions: (g?.conditions || []).map(c =>
              c?.id === conditionId ? { ...c, operator } : c
            ),
          };
        }
        return g;
      });
      emitUpdate({ ...props.config, groups });
    };

    const updateConditionValue = (groupId, conditionId, value) => {
      const groups = (props.config?.groups || []).map(g => {
        if (g?.id === groupId) {
          return {
            ...g,
            conditions: (g?.conditions || []).map(c =>
              c?.id === conditionId ? { ...c, value } : c
            ),
          };
        }
        return g;
      });
      emitUpdate({ ...props.config, groups });
    };

    return {
      getFieldsForCollection,
      getOperatorsForField,
      isValueRequired,
      addGroup,
      removeGroup,
      updateGroupCollection,
      updateGroupOperator,
      addCondition,
      removeCondition,
      updateConditionField,
      updateConditionOperator,
      updateConditionValue,
    };
  },
};
</script>

<style lang="scss" scoped>
@import '../styles/polaris-weweb-styles.scss';

.polaris-card {
  @include polaris-tokens;
  @include polaris-card;
  
  &--subdued { @include polaris-card-subdued; }
}

.polaris-card__section { 
  @include polaris-card-section;
  
  > * + * { margin-top: var(--p-space-300); }
}

.polaris-inline {
  @include polaris-inline;
  &--space-between { justify-content: space-between; }
  &--gap-tight { gap: var(--p-space-200); }
}

.polaris-block-stack {
  @include polaris-block-stack;
  gap: var(--p-space-300);
  &--tight { gap: var(--p-space-200); }
}

.polaris-text {
  &--body-md { @include polaris-text-body; }
  &--heading-sm { @include polaris-text-heading-sm; }
}

.polaris-button {
  @include polaris-button-base;
  &--plain { @include polaris-button-plain; }
  &--critical { color: var(--p-color-text-critical); }
  &--outline { @include polaris-button-outline; }
  &--icon-only { @include polaris-button-icon-only; }
  &--full-width { @include polaris-button-full-width; }
  &--slim { @include polaris-button-slim; }
  
  &--segmented {
    border-radius: 0;
    background: var(--p-color-bg-surface);
    color: var(--p-color-text);
    box-shadow: inset 0 0 0 1px var(--p-color-border);
    &:first-child { border-radius: var(--p-border-radius-200) 0 0 var(--p-border-radius-200); }
    &:last-child { border-radius: 0 var(--p-border-radius-200) var(--p-border-radius-200) 0; }
  }
  &--segmented-selected {
    background: var(--p-color-bg-surface-selected);
    color: var(--p-color-text-brand);
    box-shadow: inset 0 0 0 2px var(--p-color-border-brand);
  }
}

.polaris-button-group {
  @include polaris-button-group;
  &--segmented { @include polaris-button-group-segmented; }
  &--small .polaris-button { @include polaris-button-slim; }
}

.polaris-text-field {
  display: flex;
  flex-direction: column;
  gap: var(--p-space-100);
  &--flex { flex: 1; }
  &--operator { width: 120px; flex-shrink: 0; }
}

.polaris-text-field__label { @include polaris-label; }
.polaris-text-field__input { @include polaris-input; }
.polaris-select__input { @include polaris-select; }

.polaris-icon {
  @include polaris-icon;
  &--small { @include polaris-icon-small; }
}

.polaris-condition-list {
  display: flex;
  flex-direction: column;
  gap: var(--p-space-200);
  margin-top: var(--p-space-300);
}

.polaris-condition-item {
  display: flex;
  flex-direction: column;
  gap: var(--p-space-200);
}

.polaris-condition-operator {
  display: flex;
  justify-content: center;
  padding: var(--p-space-100) 0;
}

.polaris-condition-fields {
  display: flex;
  gap: var(--p-space-200);
  align-items: flex-end;
}

// Consistent spacing
.polaris-card + .polaris-card { margin-top: var(--p-space-300); }
.polaris-card--subdued + .polaris-card--subdued { margin-top: var(--p-space-300); }
.polaris-text-field + .polaris-text-field { margin-top: var(--p-space-300); }
</style>

