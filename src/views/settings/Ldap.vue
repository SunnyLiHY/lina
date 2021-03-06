<template>
  <div v-if="!loading">
    <IBox>
      <GenericCreateUpdateForm
        :fields="selectFields"
        :url="url"
        :initial="initialData"
        :update-success-next-route="successUrl"
        :clean-form-value="cleanFormValue"
        :object="initialData"
        :fields-meta="fieldsMeta"
        :get-method="getMethod"
        :more-buttons="moreButtons"
      />
    </IBox>
    <el-dialog :visible.sync="dialogVisible" center>
      <div slot="title">
        {{ $t('setting.testLdapLoginTitle') }}
        <br>
        <small>
          {{ $t('setting.testLdapLoginSubtitle') }}
        </small>
      </div>
      <el-form
        :model="userLoginForm"
        label-position="right"
        label-width="17%"
      >
        <el-form-item :label="$t('setting.username')">
          <el-input
            v-model="userLoginForm.username"
            :placeholder="$t('setting.usernamePlaceholder')"
            autocomplete="off"
          />

        </el-form-item>
        <el-form-item :label="$t('setting.password')">
          <el-input
            v-model="userLoginForm.password"
            type="password"
            :placeholder="$t('setting.passwordPlaceholder')"
            autocomplete="off"
          />
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="dialogVisible = false">{{ $t('common.Cancel') }}</el-button>
        <el-button type="primary" @click="testUerLoginClick">{{ $t('common.Confirm') }}</el-button>
      </div>
    </el-dialog>
    <el-dialog :visible.sync="dialogLdapUserImport" center>
      <div slot="title">
        {{ $t('setting.importLdapUserTitle') }}
        <el-alert type="success"> {{ $t('setting.importLdapUserTip') }}</el-alert>
      </div>
      <ListTable
        ref="listTable"
        :table-config="tableConfig"
        :header-actions="headerActions"
        @error="handlerListTableXHRError($event)"
      />
      <div slot="footer">
        <el-button @click="dialogLdapUserImport = false">{{ $t('common.Cancel') }}</el-button>
        <el-button type="primary" @click="importUserClick">{{ $t('common.Import') }}</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import GenericCreateUpdateForm from '@/layout/components/GenericCreateUpdateForm'
import { testLdapSetting, testLdapUserLogin,
  importLdapUser, refreshLdapUserCache, StartLdapUserCache } from '@/api/settings'
import ListTable from '@/components/ListTable'
import { IBox } from '@/components'

export default {
  name: 'Ldap',
  components: {
    GenericCreateUpdateForm,
    ListTable,
    IBox
  },
  props: {
    object: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      loading: true,
      dialogVisible: false,
      dialogLdapUserImport: false,
      initialData: {},
      selectFields: [[this.$t('common.BasicInfo'), ['AUTH_LDAP_SERVER_URI', 'AUTH_LDAP_BIND_DN', 'AUTH_LDAP_BIND_PASSWORD', 'AUTH_LDAP_SEARCH_OU',
        'AUTH_LDAP_SEARCH_FILTER', 'AUTH_LDAP_USER_ATTR_MAP', 'AUTH_LDAP']]],
      successUrl: { name: 'Settings', params: { activeMenu: 'Email' }},
      fieldsMeta: {
        AUTH_LDAP_SERVER_URI: {
          label: this.$t('setting.authLdapServerUri'),
          rules: [
            { required: true, message: this.$t('common.fieldRequiredError') }
          ]
        },
        AUTH_LDAP_BIND_DN: {
          label: this.$t('setting.authLdapBindDn')
        },
        AUTH_LDAP_BIND_PASSWORD: {
          label: this.$t('setting.authLdapBindPassword'),
          type: 'input',
          el: {
            type: 'password'
          }
        },
        AUTH_LDAP_SEARCH_OU: {
          label: this.$t('setting.authLdapSearchOu'),
          helpText: this.$t('setting.helpText.authLdapSearchOu')
        },
        AUTH_LDAP_SEARCH_FILTER: {
          label: this.$t('setting.authLdapSearchFilter'),
          rules: [
            { required: true, message: this.$t('common.fieldRequiredError') }
          ],
          helpText: this.$t('setting.helpText.authLdapSearchFilter')
        },
        AUTH_LDAP_USER_ATTR_MAP: {
          label: this.$t('setting.authLdapUserAttrMap'),
          el: {
            type: 'textarea'
          },
          rules: [
            { required: true, message: this.$t('common.fieldRequiredError') }
          ],
          helpText: this.$t('setting.helpText.authLdapUserAttrMap')
        },
        AUTH_LDAP: {
          label: this.$t('setting.authLdap'),
          type: 'checkbox'
        }
      },
      url: '/api/v1/settings/setting/',
      moreButtons: [
        {
          title: this.$t('setting.ldapConnectTest'),
          callback: function(value, form) {
            if (value['AUTH_LDAP_BIND_PASSWORD'] === undefined) {
              value['AUTH_LDAP_BIND_PASSWORD'] = ''
            }
            testLdapSetting(value).then(resp => {
              this.$message.success(resp)
            }).catch(err => {
              const response = err.response
              this.$message.error(response.data)
            })
          }.bind(this)
        },
        {
          title: this.$t('setting.ldapLoginTest'),
          callback: function(value, form) {
            this.dialogVisible = true
          }.bind(this)
        },
        {
          title: this.$t('setting.ldapBulkImport'),
          callback: function(value, form) {
            this.dialogLdapUserImport = true
          }.bind(this)
        }
      ],
      userLoginForm: {
        username: '',
        password: ''
      },
      tableConfig: {
        url: '/api/v1/settings/ldap/users/',
        columns: ['username', 'name', 'email', 'existing']
      },
      headerActions: {
        hasCreate: false,
        hasBulkDelete: false,
        hasUpload: false,
        hasExport: false,
        hasImport: false,
        hasUpdate: false,
        hasRefresh: false,
        extraActions: [
          {
            name: 'refresh',
            title: this.$t('setting.refreshLdapUser'),
            type: 'primary',
            has: true,
            can: true,
            callback: function() {
              refreshLdapUserCache().then(res => {
                this.$message.success(this.$t('setting.refreshLdapCache'))
                StartLdapUserCache()
              })
            }.bind(this)
          }
        ]
      }
    }
  },
  mounted() {
    Object.assign(this.initialData, this.object)
    if (this.object.AUTH_LDAP_USER_ATTR_MAP !== null) {
      this.initialData.AUTH_LDAP_USER_ATTR_MAP = JSON.stringify(this.object.AUTH_LDAP_USER_ATTR_MAP)
    }
    this.loading = false
  },
  methods: {
    cleanFormValue(data) {
      if (data['AUTH_LDAP_BIND_PASSWORD'] === '') {
        delete data['AUTH_LDAP_BIND_PASSWORD']
      }
      if (data['AUTH_LDAP_USER_ATTR_MAP']) {
        data['AUTH_LDAP_USER_ATTR_MAP'] = JSON.parse(data['AUTH_LDAP_USER_ATTR_MAP'])
      }
      return {
        ldap: data
      }
    },
    getMethod() {
      return 'put'
    },
    testUerLoginClick() {
      testLdapUserLogin(this.userLoginForm).then(res => {
        this.$message.success(res)
      }).catch(err => {
        const response = err.response
        this.$message.error(response.data)
      })
    },
    importUserClick() {
      const selectIds = []
      this.$refs.listTable.selectedRows.forEach((item, index) => { selectIds.push(item.id) })
      const data = {
        username_list: selectIds
      }
      importLdapUser(data).then(res => {
        this.$message.success(res.msg)
      })
    },
    handlerListTableXHRError(errMsg) {
      if (this.dialogLdapUserImport) {
        setTimeout(this.$refs.listTable.reloadTable, 30000)
      }
    }
  }
}
</script>

<style scoped>

</style>
