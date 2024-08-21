<template>
  <div class="uplink-container">
    <div style="color:#909399;margin-bottom: 30px">
      当前用户：{{ name }};
      用户角色: {{ userType }}
    </div>
    <div>
      <el-form ref="form" :model="tracedata" label-width="80px" size="mini">
        <el-form-item v-show="userType !== '原料商' && userType !== '消费者'" label="溯源码:" style="width: 300px" label-width="120px">
          <el-input v-model="tracedata.traceability_code" />
        </el-form-item>

        <template v-for="(field, index) in formFields" :key="index">
          <el-form-item :label="field.label" style="width: 300px" label-width="120px" v-show="userType === field.role">
            <el-input v-model="field.model" />
          </el-form-item>
        </template>
      </el-form>

      <span slot="footer" style="color: gray;" class="dialog-footer">
        <el-button v-show="userType !== '消费者'" type="primary" plain style="margin-left: 220px;" @click="submittracedata">提 交</el-button>
      </span>
      <span v-show="userType === '消费者'" slot="footer" style="color: gray;" class="dialog-footer">
        消费者没有权限录入！请使用溯源功能!
      </span>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import { uplink } from '@/api/trace'

export default {
  name: 'Uplink',
  data() {
    return {
      tracedata: {
        traceability_code: '',
        Farmer_input: {
          Fa_fruitName: '',
          Fa_origin: '',
          Fa_plantTime: '',
          Fa_pickingTime: '',
          Fa_farmerName: ''
        },
        Factory_input: {
          Fac_productName: '',
          Fac_productionbatch: '',
          Fac_productionTime: '',
          Fac_factoryName: '',
          Fac_contactNumber: ''
        },
        Driver_input: {
          Dr_name: '',
          Dr_age: '',
          Dr_phone: '',
          Dr_carNumber: '',
          Dr_transport: ''
        },
        Shop_input: {
          Sh_storeTime: '',
          Sh_sellTime: '',
          Sh_shopName: '',
          Sh_shopAddress: '',
          Sh_shopPhone: ''
        }
      },
      loading: false
    }
  },
  computed: {
    ...mapGetters(['name', 'userType']),
    formFields() {
      return [
        { role: '原料商', label: '原料名称:', model: this.tracedata.Farmer_input.Fa_fruitName },
        { role: '原料商', label: '原料产地:', model: this.tracedata.Farmer_input.Fa_origin },
        { role: '原料商', label: '生产时间:', model: this.tracedata.Farmer_input.Fa_plantTime },
        { role: '原料商', label: '入库时间:', model: this.tracedata.Farmer_input.Fa_pickingTime },
        { role: '原料商', label: '原料商名称:', model: this.tracedata.Farmer_input.Fa_farmerName },
        { role: '工厂', label: '商品名称:', model: this.tracedata.Factory_input.Fac_productName },
        { role: '工厂', label: '生产批次:', model: this.tracedata.Factory_input.Fac_productionbatch },
        { role: '工厂', label: '生产时间:', model: this.tracedata.Factory_input.Fac_productionTime },
        { role: '工厂', label: '工厂名称与厂址:', model: this.tracedata.Factory_input.Fac_factoryName },
        { role: '工厂', label: '联系电话:', model: this.tracedata.Factory_input.Fac_contactNumber },
        { role: '运输司机', label: '姓名:', model: this.tracedata.Driver_input.Dr_name },
        { role: '运输司机', label: '年龄:', model: this.tracedata.Driver_input.Dr_age },
        { role: '运输司机', label: '联系电话:', model: this.tracedata.Driver_input.Dr_phone },
        { role: '运输司机', label: '车牌号:', model: this.tracedata.Driver_input.Dr_carNumber },
        { role: '运输司机', label: '运输工具:', model: this.tracedata.Driver_input.Dr_transport },
        { role: '商店', label: '存入时间:', model: this.tracedata.Shop_input.Sh_storeTime },
        { role: '商店', label: '销售时间:', model: this.tracedata.Shop_input.Sh_sellTime },
        { role: '商店', label: '商店名称:', model: this.tracedata.Shop_input.Sh_shopName },
        { role: '商店', label: '商店位置:', model: this.tracedata.Shop_input.Sh_shopAddress },
        { role: '商店', label: '商店电话:', model: this.tracedata.Shop_input.Sh_shopPhone }
      ]
    }
  },
  methods: {
    submittracedata() {
      console.log(this.tracedata)
      const loading = this.$loading({
        lock: true,
        text: '数据上链中...',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      })

      const formData = new FormData()

      if (this.userType !== '原料商') {
        formData.append('traceability_code', this.tracedata.traceability_code)
      }

      const inputFields = this.formFields.filter(field => field.role === this.userType)
      inputFields.forEach((field, index) => {
        formData.append(`arg${index + 1}`, field.model)
      })

      uplink(formData)
        .then(res => {
          loading.close()
          if (res.code === 200) {
            this.$message({
              message: '上链成功，交易ID：' + res.txid + '\n溯源码：' + res.traceability_code,
              type: 'success'
            })
          } else {
            this.$message({
              message: '上链失败',
              type: 'error'
            })
          }
        })
        .catch(err => {
          loading.close()
          console.log(err)
        })
    }
  }
}
</script>

<style lang="scss" scoped>
.uplink {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
</style>
