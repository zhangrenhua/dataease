<template>
  <div>
    <el-form ref="createOrganization" inline :model="form" size="small" label-width="80px">

      <el-form-item ref="form" :label="$t('panel.link_share')">
        <el-switch
          v-model="valid"
          style="width: 370px;"
          :active-value="true"
          :inactive-value="false"
          @change="onChange"
        />
      </el-form-item>
      <el-form-item label=" ">
        <el-link class="de-link" style="width: 370px;" disabled>{{ $t('panel.link_share_desc') }}</el-link>
      </el-form-item>
      <el-form-item v-if="valid" :label="$t('panel.link')">
        <el-input
          v-model.number="form.uri"
          disabled
          style="width: 370px;"
        />
      </el-form-item>

      <el-form-item v-if="valid" :label="$t('panel.over_time')">
        <el-date-picker
          v-model="form.overTime"
          type="datetime"
          placeholder="选择日期时间"
          align="right"
          value-format="timestamp"
          :picker-options="pickerOptions"
          default-time="23:59:59"
          popper-class="link-date-picker-class"
          @change="resetOverTime"
        />
      </el-form-item>

      <el-form-item v-if="valid" label=" ">
        <el-checkbox v-model="form.enablePwd" @change="resetEnablePwd">{{ $t('panel.passwd_protect') }}  </el-checkbox>

        <span v-if="form.enablePwd" class="de-span">{{ form.pwd }}</span>
        <span v-if="form.enablePwd" class="de-span" @click="resetPwd"><el-link :underline="false" type="primary">{{ $t('commons.reset') }}</el-link></span>
      </el-form-item>

      <div v-if="valid" class="auth-root-class">
        <span slot="footer">

          <el-button v-if="!form.enablePwd" v-clipboard:copy="form.uri" v-clipboard:success="onCopy" v-clipboard:error="onError" size="mini" type="primary">{{ $t('panel.copy_link') }}</el-button>
          <el-button v-if="form.enablePwd" v-clipboard:copy="form.uri + ' Password: '+ form.pwd" v-clipboard:success="onCopy" v-clipboard:error="onError" size="mini" type="primary">{{ $t('panel.copy_link_passwd') }}</el-button>

        </span>
      </div>

    </el-form>
  </div>
</template>
<script>

import { loadGenerate, setPwd, switchValid, switchEnablePwd, shortUrl, setOverTime } from '@/api/link'
export default {

  name: 'LinkGenerate',
  components: {

  },
  props: {
    resourceId: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      pwdNums: 4,
      valid: false,
      form: {},
      defaultForm: { enablePwd: false, pwd: null, uri: null },
      pickerOptions: {
        disabledDate: time => {
          return time < (Date.now() - 8.64e7)
        },
        shortcuts: [{
          text: '一天',
          onClick: function(picker) {
            picker.$emit('pick', this.limitDate('day'))
          }.bind(this)
        }, {
          text: '一周',
          onClick: (picker) => {
            picker.$emit('pick', this.limitDate('week'))
          }
        }, {
          text: '一月',
          onClick: (picker) => {
            picker.$emit('pick', this.limitDate('month'))
          }
        }]
      }
    }
  },
  computed: {
    origin() {
      return window.location.origin
    }
  },
  created() {
    this.form = this.defaultForm
    this.currentGenerate()
  },
  methods: {
    currentGenerate() {
      loadGenerate(this.resourceId).then(res => {
        const { valid, enablePwd, pwd, uri, overTime } = res.data
        this.valid = valid
        this.form.enablePwd = enablePwd
        this.form.uri = uri ? (this.origin + uri) : uri
        // 返回的密码是共钥加密后的 所以展示需要私钥解密一波
        pwd && (this.form.pwd = pwd)

        if (pwd && pwd.length > 0 && pwd.length > this.pwdNums) {
          this.resetPwd()
        }
        /* pwd && (this.form.pwd = decrypt(pwd)) */
        /* overTime && (this.form.overTime = overTime) */
        overTime && (this.$set(this.form, 'overTime', overTime))
        this.requestShort()
      })
    },

    createPwd() {
      const randomNum = () => {
        return Math.floor(Math.random() * 10) + ''
      }
      let result = ''
      for (let index = 0; index < this.pwdNums; index++) {
        result += randomNum()
      }
      return result
    },

    resetPwd() {
      // 密码采用RSA共钥加密
      const newPwd = this.createPwd()
      const param = {
        resourceId: this.resourceId,
        password: newPwd
        /* password: encrypt(newPwd) */
      }
      setPwd(param).then(res => {
        this.form.pwd = newPwd
      })
    },
    resetEnablePwd(value) {
      const param = {
        resourceId: this.resourceId,
        enablePwd: value
      }
      switchEnablePwd(param).then(res => {
        // 当切换到启用密码保护 如果没有密码 要生成密码
        value && !this.form.pwd && this.resetPwd()
      })
    },
    resetOverTime(value) {
      const param = {
        resourceId: this.resourceId,
        overTime: value
      }
      setOverTime(param).then(res => {
        // this.form.overTime = value
        this.$forceUpdate()
      })
    },

    onCopy(e) {
      this.$success(this.$t('commons.copy_success'))
    },
    onError(e) {
    },
    onChange(value) {
      const param = {
        resourceId: this.resourceId,
        valid: value
      }
      switchValid(param).then(res => {

      })
    },
    requestShort() {
      const url = this.form.uri
      if (!url) return

      shortUrl({ resourceId: this.resourceId }).then(res => {
        if (res.success) {
          this.form.uri = this.origin + res.data
        }
      })
    },

    limitDate(type) {
      const now = new Date()
      const nowTime = now.getTime()
      const oneDay = 24 * 60 * 60 * 1000

      if (type === 'day') {
        const tom = new Date(nowTime + oneDay)
        return new Date(tom.format('yyyy-MM-dd') + ' 23:59:59')
      }
      if (type === 'week') {
        const tom = new Date(nowTime + oneDay * 7)
        return new Date(tom.format('yyyy-MM-dd') + ' 23:59:59')
      }
      if (type === 'month') {
        const result = new Date()
        const curMonth = now.getMonth() + 1
        if (curMonth === 12) {
          result.setYear(now.getYear() + 1)
          result.setMonth(0)
        } else {
          result.setMonth(curMonth)
        }
        return new Date(result.format('yyyy-MM-dd') + ' 23:59:59')
      }
      return null
    }
  }
}
</script>

<style lang="scss" scoped>
    .de-link{
        justify-content: left !important;
    }
    .de-span {
        margin: 0 15px;
    }
    .auth-root-class {
        margin: 15px 0px 5px;
        text-align: right;
    }

</style>
