<template>
  <b-card
    v-if="resource"
    data-test-id="card-auth-client-info"
    class="shadow-sm auth-clients"
    header-bg-variant="white"
    footer-bg-variant="white"
  >
    <b-form
      @submit.prevent="submit"
    >
      <b-form-group
        :label="$t('name')"
        label-cols="3"
      >
        <b-form-input
          v-model="resource.meta.name"
          data-test-id="input-name"
          required
          :state="nameState"
        />
      </b-form-group>

      <b-form-group
        :label="$t('handle.label')"
        label-cols="3"
      >
        <b-form-input
          v-model="resource.handle"
          data-test-id="input-handle"
          :disabled="resource.isDefault"
          :placeholder="$t('handle.placeholder-handle')"
          :state="handleState"
        />
        <b-form-invalid-feedback
          data-test-id="feedback-invalid-handle"
          :state="handleState"
        >
          {{ $t('handle.invalid-handle-characters') }}
        </b-form-invalid-feedback>
        <template
          v-if="resource.isDefault"
          #description
        >
          {{ $t('handle.disabledFootnote') }}
        </template>
      </b-form-group>

      <b-form-group
        :label="$t('redirectURI')"
        label-cols="3"
      >
        <b-button
          data-test-id="button-add-redirect-uris"
          variant="light"
          class="align-top"
          @click="redirectURI.push('')"
        >
          + {{ $t('add') }}
        </b-button>

        <div
          v-if="redirectURI.length"
        >
          <b-input-group
            v-for="(value, index) in redirectURI"
            :key="index"
            class="mt-2"
          >
            <b-form-input
              v-model="redirectURI[index]"
              data-test-id="input-uri"
              :placeholder="$t('uri')"
            />

            <b-button
              data-test-id="button-remove-uri"
              class="ml-1 text-danger"
              variant="link"
              @click="redirectURI.splice(index, 1)"
            >
              <font-awesome-icon
                :icon="['fas', 'times']"
              />
            </b-button>
          </b-input-group>
        </div>
      </b-form-group>

      <b-form-group
        v-if="!fresh"
        :label="$t('secret')"
        label-cols="3"
        class="mb-3"
      >
        <div class="d-flex">
          <b-form-input
            v-model="secret"
            data-test-id="input-client-secret"
            disabled
            placeholder="****************************************************************"
          />

          <b-button
            v-if="!secretVisible"
            data-test-id="button-show-client-secret"
            class="text-primary border-0 px-3"
            variant="outline-light"
            @click="$emit('request-secret')"
          >
            <font-awesome-icon
              :icon="['fas', 'eye']"
            />
          </b-button>

          <b-button
            v-else
            data-test-id="button-regenerate-client-secret"
            class="text-primary border-0 px-3"
            variant="outline-light"
            :title="$t('tooltip.regenerate-secret')"
            @click="$emit('regenerate-secret')"
          >
            <font-awesome-icon
              :icon="['fas', 'sync']"
            />
          </b-button>
        </div>
      </b-form-group>

      <b-form-group
        label-cols="3"
      >
        <b-form-radio-group
          v-model="resource.validGrant"
          value="authorization_code"
          :options="[
            { value: 'authorization_code', text: $t('grant.authorization_code') },
            { value: 'client_credentials', text: $t('grant.client_credentials') },
          ]"
        />
      </b-form-group>

      <b-form-group
        data-test-id="valid-from"
        :label="$t('validFrom.label')"
        label-cols="3"
        :description="$t('validFrom.description')"
      >
        <c-input-date-time
          v-model="resource.validFrom"
          data-test-id="input-valid-from"
          :labels="{
            clear: $t('general:label.clear'),
            none: $t('general:label.none'),
            now: $t('general:label.now'),
            today: $t('general:label.today'),
          }"
        />
      </b-form-group>

      <b-form-group
        data-test-id="expires-at"
        :label="$t('expiresAt.label')"
        label-cols="3"
        :description="$t('expiresAt.description')"
      >
        <c-input-date-time
          v-model="resource.expiresAt"
          data-test-id="input-expires-at"
          :labels="{
            clear: $t('general:label.clear'),
            none: $t('general:label.none'),
            now: $t('general:label.now'),
            today: $t('general:label.today'),
          }"
        />
      </b-form-group>

      <b-form-group
        label-cols="3"
      >
        <b-form-checkbox
          data-test-id="checkbox-allow-access-to-user-profile"
          :checked="(resource.scope || []).includes('profile')"
          @change="setScope($event, 'profile')"
        >
          {{ $t('profile') }}
        </b-form-checkbox>
        <b-form-checkbox
          data-test-id="checkbox-allow-access-to-corteza-api"
          :checked="(resource.scope || []).includes('api')"
          @change="setScope($event, 'api')"
        >
          {{ $t('api') }}
        </b-form-checkbox>
        <b-form-checkbox
          data-test-id="checkbox-allow-client-to-use-oidc"
          :checked="(resource.scope || []).includes('openid')"
          @change="setScope($event, 'openid')"
        >
          {{ $t('openid') }}
        </b-form-checkbox>
        <b-form-checkbox
          v-if="discoveryEnabled"
          data-test-id="checkbox-allow-client-access-to-discovery"
          :checked="(resource.scope || []).includes('discovery')"
          @change="setScope($event, 'discovery')"
        >
          {{ $t('discovery') }}
        </b-form-checkbox>
      </b-form-group>

      <b-form-group
        label-cols="3"
      >
        <b-form-checkbox
          v-model="resource.trusted"
          data-test-id="checkbox-is-client-trusted"
        >
          {{ $t('trusted.label') }}
        </b-form-checkbox>
        <b-form-text>{{ $t('trusted.description') }}</b-form-text>
      </b-form-group>

      <b-form-group
        label-cols="3"
      >
        <b-form-checkbox
          v-model="resource.enabled"
          data-test-id="checkbox-is-client-enabled"
          :disabled="resource.isDefault"
        >
          {{ $t('enabled.label') }}
        </b-form-checkbox>

        <template
          v-if="resource.isDefault"
          #description
        >
          {{ $t('enabled.disabledFootnote') }}
        </template>
      </b-form-group>

      <div v-if="isClientCredentialsGrant">
        <b-form-group
          data-test-id="impersonate-user"
          label-cols="3"
          :label="$t('security.impersonateUser.label')"
          :description="$t('security.impersonateUser.description')"
        >
          <c-select-user
            :user-i-d="resource.security.impersonateUser"
            @updateUser="onUpdateUser"
          />
        </b-form-group>
        <div v-if="!fresh">
          <b-form-group label-cols="3">
            <b-button
              variant="light"
              class="align-top"
              @click="toggleCurlSnippet()"
            >
              <template v-if="curlVisible">
                {{ $t('hideCurl') }}
              </template>
              <template v-else>
                {{ $t('generateCurl') }}
              </template>
            </b-button>
          </b-form-group>
          <b-form-group
            v-if="curlVisible"
            :label="$t('cUrl')"
            label-cols="3"
            class="curl"
          >
            <div class="w-100">
              <div class="d-flex">
                <pre
                  ref="cUrl"
                  data-test-id="cURL"
                  style="word-break: break-word;"
                >
curl -X POST {{ curlURL }} \
-d grant_type=client_credentials \
-d scope='profile api' \
-u {{ resource.authClientID }}:{{ secret || 'PLACE-YOUR-CLIENT-SECRET-HERE' }}
                </pre>
                <b-button
                  data-test-id="copy-cURL"
                  variant="link"
                  class="align-top ml-auto fit-content text-secondary mr-5"
                  @click="copyToClipboard('cUrl')"
                >
                  <font-awesome-icon
                    :icon="['far', 'copy']"
                  />
                </b-button>
              </div>
              <div class="d-flex">
                <div
                  class="overflow-wrap mr-2 mb-2"
                  :class="[tokenRequest.token ? 'text-success' : 'text-danger']"
                >
                  {{ tokenRequest.token || tokenRequest.error }}
                </div>
                <b-button
                  v-if="tokenRequest.token"
                  data-test-id="copy-token-from-request"
                  variant="link"
                  class="align-top ml-auto fit-content text-secondary"
                  @click="copyToClipboard('token')"
                >
                  <font-awesome-icon
                    :icon="['far', 'copy']"
                  />
                </b-button>
              </div>
            </div>
            <div
              v-if="secretVisible"
              class="d-flex mb-3"
            >
              <b-button
                data-test-id="button-test-cURL"
                variant="light"
                class="align-top fit-content"
                @click="getAccessTokenAPI()"
              >
                {{ $t('testCurl') }}
              </b-button>
            </div>
          </b-form-group>
        </div>
      </div>

      <b-form-group
        data-test-id="permitted-roles"
        :label="$t('security.permittedRoles.label')"
        label-cols="3"
        class="mb-0"
      >
        <c-role-picker
          v-model="resource.security.permittedRoles"
          class="mb-3"
        >
          <template #description>
            {{ $t('security.permittedRoles.description') }}
          </template>
        </c-role-picker>
      </b-form-group>

      <b-form-group
        :label="$t('security.prohibitedRoles.label')"
        data-test-id="prohibited-roles"
        label-cols="3"
        class="mb-0"
      >
        <c-role-picker
          v-model="resource.security.prohibitedRoles"
          class="mb-3"
        >
          <template #description>
            {{ $t('security.prohibitedRoles.description') }}
          </template>
        </c-role-picker>
      </b-form-group>

      <b-form-group
        data-test-id="forced-roles"
        :label="$t('security.forcedRoles.label')"
        label-cols="3"
        class="mb-0"
      >
        <c-role-picker
          v-model="resource.security.forcedRoles"
          class="mb-3"
        >
          <template #description>
            {{ $t('security.forcedRoles.description') }}
          </template>
        </c-role-picker>
      </b-form-group>

      <b-form-group
        v-if="resource.createdAt"
        :label="$t('createdAt')"
        label-cols="3"
        class="mb-0"
      >
        <b-form-input
          data-test-id="created-at"
          :value="resource.createdAt | locFullDateTime"
          plaintext
          disabled
        />
      </b-form-group>

      <b-form-group
        v-if="resource.updatedAt"
        :label="$t('updatedAt')"
        label-cols="3"
      >
        <b-form-input
          data-test-id="updated-at"
          :value="resource.updatedAt | locFullDateTime"
          plaintext
          disabled
        />
      </b-form-group>

      <b-form-group
        v-if="resource.deletedAt"
        :label="$t('deletedAt')"
        label-cols="3"
      >
        <b-form-input
          data-test-id="deleted-at"
          :value="resource.deletedAt | locFullDateTime"
          plaintext
          disabled
        />
      </b-form-group>

      <!--
        include hidden input to enable
        trigger submit event w/ ENTER
      -->
      <input
        data-test-id="button-submit"
        type="submit"
        class="d-none"
        :disabled="saveDisabled"
      >
    </b-form>

    <template #header>
      <h3 class="m-0">
        {{ $t('title') }}
      </h3>
    </template>

    <template #footer>
      <c-submit-button
        class="float-right"
        :disabled="saveDisabled"
        :processing="processing"
        :success="success"
        @submit="submit"
      />

      <template
        v-if="canDelete"
      >
        <confirmation-toggle
          v-if="isDeleted"
          data-test-id="button-undelete"
          :disabled="processing"
          @confirmed="$emit('undelete', resource.authClientID)"
        >
          {{ $t('undelete') }}
        </confirmation-toggle>
        <confirmation-toggle
          v-else
          data-test-id="button-delete"
          :disabled="processing"
          @confirmed="$emit('delete', resource.authClientID)"
        >
          {{ $t('delete') }}
        </confirmation-toggle>
      </template>
    </template>
  </b-card>
</template>

<script>
import { NoID } from '@cortezaproject/corteza-js'
import { handle, components } from '@cortezaproject/corteza-vue'
import ConfirmationToggle from 'corteza-webapp-admin/src/components/ConfirmationToggle'
import CSubmitButton from 'corteza-webapp-admin/src/components/CSubmitButton'
import CRolePicker from 'corteza-webapp-admin/src/components/CRolePicker'
import CSelectUser from 'corteza-webapp-admin/src/components/Authclient/CSelectUser'
import copy from 'copy-to-clipboard'
import axios from 'axios'

const { CInputDateTime } = components

export default {
  name: 'CAuthclientEditorInfo',

  i18nOptions: {
    namespaces: 'system.authclients',
    keyPrefix: 'editor.info',
  },

  components: {
    ConfirmationToggle,
    CSubmitButton,
    CRolePicker,
    CSelectUser,
    CInputDateTime,
  },

  props: {
    resource: {
      type: Object,
      required: true,
    },

    canDelete: {
      type: Boolean,
      default: () => false,
    },

    processing: {
      type: Boolean,
      value: false,
    },

    secret: {
      type: String,
      default: () => '',
    },

    success: {
      type: Boolean,
      value: false,
    },

    canCreate: {
      type: Boolean,
      required: true,
    },
  },

  data () {
    return {
      redirectURI: this.resource.redirectURI ? this.resource.redirectURI.split(' ') : [],

      curlVisible: false,
      curlURL: '',
      tokenRequest: {
        token: '',
        error: '',
      },
    }
  },

  computed: {
    fresh () {
      return !this.resource.authClientID || this.resource.authClientID === NoID
    },

    editable () {
      return this.fresh ? this.canCreate : this.resource.canUpdateAuthClient
    },

    isDeleted () {
      return this.resource.deletedAt && this.resource.canDeleteAuthClient
    },

    secretVisible () {
      return this.secret.length > 0
    },

    nameState () {
      return this.resource.meta.name ? null : false
    },

    handleState () {
      return handle.handleState(this.resource.handle)
    },

    isClientCredentialsGrant () {
      return this.resource.validGrant === 'client_credentials'
    },

    discoveryEnabled () {
      return this.$Settings.get('discovery.enabled', false)
    },

    saveDisabled () {
      return !this.editable || [this.nameState, this.handleState].includes(false)
    },
  },

  watch: {
    'redirectURI': {
      handler (redirectURI) {
        this.resource.redirectURI = redirectURI.filter(ru => ru).join(' ')
      },
    },
  },

  methods: {
    onUpdateUser (user) {
      this.resource.security.impersonateUser = (user || {}).userID
    },

    getAccessTokenAPI () {
      const params = new URLSearchParams()
      params.append('grant_type', 'client_credentials')
      params.append('scope', 'profile api')
      axios.post(
        this.curlURL,
        params,
        { auth: { username: this.resource.authClientID, password: this.secret } }
      ).then(response => {
        this.tokenRequest.token = (response.data || {}).access_token
      }).catch(error => {
        this.tokenRequest.error = error
      })
    },

    copyToClipboard (name) {
      if (name === 'cUrl') {
        copy(this.$refs.cUrl.innerHTML)
      } else {
        copy(this.tokenRequest.token)
      }
    },

    toggleCurlSnippet () {
      if (!this.curlVisible) {
        this.curlURL = this.$auth.cortezaAuthURL + '/oauth2/token'
      }
      this.curlVisible = !this.curlVisible
    },

    submit () {
      if (!this.isClientCredentialsGrant || !this.resource.security.impersonateUser) {
        this.resource.security.impersonateUser = '0'
      }

      this.$emit('submit', this.resource)
    },

    setScope (value, target) {
      let items = this.resource.scope ? this.resource.scope.split(' ') : []

      if (value) {
        items.push(target)
      } else {
        items = items.filter(i => i !== target)
      }

      this.resource.scope = items.join(' ')
    },
  },
}
</script>
<style lang="scss">
.auth-clients {
  .fit-content {
    height:fit-content;
  }
  .overflow-wrap {
      overflow-wrap: anywhere;
  }
  .curl .form-row {
    flex-wrap: nowrap !important;
    .col {
      max-width: 84.3%;
    }
  }
}
</style>
