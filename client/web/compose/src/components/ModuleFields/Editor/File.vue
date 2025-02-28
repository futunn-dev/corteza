<template>
  <b-form-group
    :label-cols-md="horizontal && '5'"
    :label-cols-xl="horizontal && '4'"
    :content-cols-md="horizontal && '7'"
    :content-cols-xl="horizontal && '8'"
    :class="formGroupStyleClasses"
  >
    <template
      #label
    >
      <div
        v-if="!valueOnly"
        class="d-flex align-items-center text-primary p-0"
      >
        <span
          class="d-inline-block mw-100 py-1"
          :title="label"
        >
          {{ label }}
        </span>

        <hint
          :id="field.fieldID"
          :text="hint"
        />

        <slot name="tools" />
      </div>
      <div
        class="small text-muted"
        :class="{ 'mb-1': description }"
      >
        {{ description }}
      </div>
    </template>

    <uploader
      :endpoint="endpoint"
      :accepted-files="mimetypes"
      :max-filesize="maxSize"
      :form-data="uploaderFormData"
      @uploaded="appendAttachment"
    />

    <list-loader
      kind="record"
      :set.sync="set"
      :namespace="namespace"
      :enable-order="field.isMulti"
      enable-delete
      mode="list"
      class="mt-2"
    />
    <errors :errors="errors" />
  </b-form-group>
</template>
<script>
import base from './base'
import Uploader from 'corteza-webapp-compose/src/components/Public/Page/Attachment/Uploader'
import ListLoader from 'corteza-webapp-compose/src/components/Public/Page/Attachment/ListLoader'
import { NoID } from '@cortezaproject/corteza-js'

export default {
  components: {
    Uploader,
    ListLoader,
  },

  extends: base,

  computed: {
    endpoint () {
      const { moduleID, recordID } = this.record
      const { namespaceID } = this.namespace

      return this.$ComposeAPI.recordUploadEndpoint({
        namespaceID,
        moduleID,
        recordID,
        fieldName: this.field.name,
      })
    },

    uploaderFormData () {
      const fd = {
        fieldName: this.field.name,
      }

      if (this.record && this.record.recordID !== NoID) {
        fd.recordID = this.record.recordID
      }

      return fd
    },

    mimetypes () {
      const a = (this.field.options.mimetypes || '').trim()
      if (!a) {
        return this.$s('compose.Record.Attachments.Mimetypes', ['*/*'])
      }

      return a.split(',').map(p => p.trim())
    },

    maxSize () {
      return this.field.options.maxSize || this.$s('compose.Record.Attachments.MaxSize', 100)
    },

    set: {
      get () {
        return this.field.isMulti ? this.value : [this.value]
      },

      set (v) {
        if (this.field.isMulti) {
          this.value = v
        } else {
          this.value = (Array.isArray(v) && v.length > 0) ? v[0] : undefined
        }
      },
    },
  },

  methods: {
    appendAttachment ({ attachmentID } = {}) {
      if (this.field.isMulti) {
        this.value.push(attachmentID)
      } else {
        this.value = attachmentID
      }
    },
  },
}
</script>
