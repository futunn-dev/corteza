<template>
  <div class="table-responsive">
    <b-row class="header pl-3">
      <b-col
        v-for="(field, index) in fields"
        :key="index"
        :class="`py-2 ${field.thClass}`"
      >
        <label>{{ field.label }}</label>
      </b-col>
    </b-row>

    <draggable
      :list="items"
      @end="$root.$emit('change-detected')"
    >
      <div
        v-for="(item, index) in items"
        :key="index"
      >
        <b-row
          class="d-flex justify-content-between align-items-center pointer expr-item"
          no-gutters
          @click="$set(item, '_showDetails', !item._showDetails)"
        >
          <b-col
            v-for="(field, i) in fields"
            :key="i"
            class="text-truncate p-2"
          >
            <div
              v-if="field.key === 'expr'"
              class="d-flex justify-content-between align-items-center"
            >
              <samp class="text-truncate">{{ item[field.key] }}</samp>

              <b-button
                v-if="item._showDetails"
                variant="outline-danger"
                size="sm"
                class="border-0"
                @click="$emit('remove', index)"
              >
                <font-awesome-icon
                  :icon="['far', 'trash-alt']"
                />
              </b-button>
            </div>

            <var
              v-else
              class=""
            >
              {{ field.formatter ? field.formatter(item) : item[field.key] }}
            </var>
          </b-col>
        </b-row>

        <transition name="fade">
          <div
            v-if="item._showDetails"
            class="mb-3 px-3"
          >
            <div class="arrow-up" />

            <b-card
              class="bg-light"
              body-class="px-4 pb-3"
            >
              <b-form-group
                label-class="text-primary"
              >
                <b-form-input
                  v-model="item.target"
                  placeholder="Target"
                  @input="$root.$emit('change-detected')"
                />
              </b-form-group>

              <b-form-group
                label-class="text-primary"
                :description="getTypeDescription(item.type)"
              >
                <vue-select
                  v-model="item.type"
                  :options="types"
                  :get-option-key="getOptionKey"
                  :clearable="false"
                  :filter="varFilter"
                  append-to-body
                  :calculate-position="calculateDropdownPosition"
                  @input="$root.$emit('change-detected')"
                />
              </b-form-group>

              <b-form-group
                class="mb-0"
              >
                <expression-editor
                  :value.sync="item[valueField]"
                  lang="javascript"
                  show-line-numbers
                  @open="$emit('open-editor', index)"
                  @input="$root.$emit('change-detected')"
                />
              </b-form-group>
            </b-card>
          </div>
        </transition>
      </div>
    </draggable>
  </div>
</template>

<script>
import ExpressionEditor from './ExpressionEditor.vue'
import { objectSearchMaker } from '../lib/filter'
import { VueSelect } from 'vue-select'
import draggable from 'vuedraggable'

export default {
  components: {
    ExpressionEditor,
    VueSelect,
    draggable,
  },

  props: {
    valueField: {
      type: String,
      required: true,
    },

    items: {
      type: Array,
      required: true,
    },

    fields: {
      type: Array,
      required: true,
    },

    types: {
      type: Array,
      required: true,
    },
  },

  methods: {
    varFilter: objectSearchMaker('text'),

    getTypeDescription (type) {
      // This will be moved to backend field type information
      const typeDescriptions = {
        ID: 'Make sure to provide the ID in double quotes if you\'re using a literal value. Example "123"',
      }

      return typeDescriptions[type]
    },

    getOptionKey (type) {
      return type
    },
  },
}
</script>

<style lang="scss" scoped>
.header {
  background-color: #d8dfe3;

  &:hover {
    background-color: #c9d3d8;
  }

  label {
    margin: 0;
  }
}

.table-responsive {
  overflow: hidden;
}

.expr-item:hover {
  background-color: #F3F3F5;
}
</style>
