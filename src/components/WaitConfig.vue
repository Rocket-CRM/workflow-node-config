<template>
  <div class="polaris-card">
    <div class="polaris-card__section">
      <div class="polaris-block-stack">
        <div class="polaris-inline polaris-inline--gap-tight">
          <div class="polaris-text-field polaris-text-field--flex">
            <label class="polaris-text-field__label polaris-text-field__label--required">Duration</label>
            <input
              type="number"
              class="polaris-text-field__input"
              :value="config?.duration || ''"
              @input="updateDuration($event.target.value)"
              min="1"
              placeholder="0"
            />
          </div>

          <div class="polaris-text-field polaris-text-field--flex">
            <label class="polaris-text-field__label polaris-text-field__label--required">Unit</label>
            <select
              class="polaris-select__input"
              :value="config?.unit || 'days'"
              @change="updateUnit($event.target.value)"
            >
              <option value="minutes">Minutes</option>
              <option value="hours">Hours</option>
              <option value="days">Days</option>
              <option value="weeks">Weeks</option>
              <option value="months">Months</option>
            </select>
          </div>
        </div>

        <div class="polaris-banner polaris-banner--info">
          <svg viewBox="0 0 20 20" class="polaris-banner__icon">
            <path d="M10 0C4.486 0 0 4.486 0 10s4.486 10 10 10 10-4.486 10-10S15.514 0 10 0zm1 15H9v-2h2v2zm0-4H9V5h2v6z" fill="currentColor"/>
          </svg>
          <div class="polaris-banner__content">
            <p class="polaris-banner__message">{{ config?.label || 'Wait [duration] [unit]' }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'WaitConfig',
  props: {
    config: { type: Object, required: true },
  },
  emits: ['update'],
  setup(props, { emit }) {
    const updateDuration = (value) => {
      const duration = parseInt(value) || 0;
      const unit = props.config?.unit || 'days';
      const label = duration > 0 ? `Wait ${duration} ${unit}` : 'Wait';
      emit('update', { ...props.config, duration, label });
    };

    const updateUnit = (unit) => {
      const duration = props.config?.duration || 1;
      const label = `Wait ${duration} ${unit}`;
      emit('update', { ...props.config, unit, label });
    };

    return {
      updateDuration,
      updateUnit,
    };
  },
};
</script>

<style lang="scss" scoped>
@import 'polaris-weweb-styles';

.polaris-card {
  @include polaris-tokens;
  @include polaris-card;
}

.polaris-card__section { @include polaris-card-section; }

.polaris-block-stack { 
  @include polaris-block-stack;
  gap: var(--p-space-300);
}

.polaris-inline {
  @include polaris-inline;
  &--gap-tight { gap: var(--p-space-200); }
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

.polaris-text-field__input { @include polaris-input; }
.polaris-select__input { @include polaris-select; }

.polaris-banner {
  @include polaris-banner-info;
}

.polaris-banner__icon { @include polaris-icon; flex-shrink: 0; }
.polaris-banner__content { flex: 1; }
.polaris-banner__message { @include polaris-text-body; margin: 0; }
</style>

