<template>
  <div>

    <div class="app-container">
      <el-form ref="userEditForm" :model="userEditForm" :rules="userEditRules" class="login-form" auto-complete="on" label-position="left">

        <div class="title-container">
          <h3 class="title">User Edit</h3>
        </div>

        <el-row :gutter="30">
          <el-col :span="12">
            <el-form-item label="Name" prop="name">
              <el-input
                ref="name"
                v-model="userEditForm.name"
                placeholder="Name"
                name="name"
                type="text"
                tabindex="1"
                auto-complete="on"
              />
            </el-form-item>
          </el-col>

          <el-col :span="12">
            <el-form-item prop="last_name" label="Last Name">
              <el-input
                ref="last_name"
                v-model="userEditForm.last_name"
                placeholder="Last Name"
                name="last_name"
                type="text"
                tabindex="1"
                auto-complete="on"
              />
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="30">
          <el-col :span="12">
            <el-form-item label="Gender" prop="gender">
              <el-select v-model="userEditForm.gender" placeholder="please select your gender">
                <el-option label="Male" value="1" />
                <el-option label="Female" value="2" />
              </el-select>
            </el-form-item>
          </el-col>

          <el-col :span="12">
            <el-form-item prop="enable" label="Enable/Disable">
              <el-switch v-model="userEditForm.enable" />
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="30">
          <el-col :span="12">
            <el-form-item label="Rol" prop="rol">
              <el-select v-model="userEditForm.rol_id" placeholder="please select your Rol">
                <template v-for="rol in roles">
                  <el-option :key="rol.id" :label="rol.name" :value="rol.id" />
                </template>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Coordinacion" prop="coordinacion">
              <el-select v-model="userEditForm.id_coordinacion" placeholder="please select your Rol">
                <template v-for="coordinacion in coordinaciones">
                  <el-option :key="coordinacion.id" :label="coordinacion.nombre" :value="coordinacion.id" />
                </template>
              </el-select>
            </el-form-item>
          </el-col>

        </el-row>

        <el-row :gutter="30">
          <el-col :span="24">
            <el-form-item prop="email" label="E-mail">
              <el-input
                ref="email"
                v-model="userEditForm.email"
                placeholder="Email"
                name="email"
                type="text"
                tabindex="1"
                auto-complete="on"
                disabled
              />
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="30">
          <el-col :span="24">
            <el-form-item prop="avatar" label="Avatar">
              <el-input
                ref="avatar"
                v-model="userEditForm.avatar"
                placeholder="avatar"
                name="avatar"
                type="text"
                tabindex="1"
                auto-complete="on"
              />
            </el-form-item>
          </el-col>
        </el-row>

        <el-form-item>
          <el-button type="primary" @click="onSubmit">Edit</el-button>
          <el-button @click="onCancel">Cancel</el-button>
        </el-form-item>

      </el-form>

    </div>
    <Footer :activar-bg="true" />
  </div>

</template>

<script>
import { getOne, update } from '@/api/user'
import { validEmail } from '@/utils/validate'
import { getAll } from '@/api/rol'
import { getListaOne } from '@/api/lista'
import Footer from '@/components/footer'

export default {
  components: { Footer },

  data() {
    const validateEmail = (rule, value, callback) => {
      if (!validEmail(value)) {
        callback(new Error('Please enter the correct user name'))
      } else {
        callback()
      }
    }

    return {
      id: 0,
      pageLoading: true,
      userEditForm: { },
      userEditRules: {
        email: [{ required: true, trigger: 'blur', validator: validateEmail }],
        name: [{ required: true }],
        last_name: [{ required: true }],
        avatar: [{ required: true }],
        gender: [{ required: true }],
        enable: [{ required: true }]
      },
      coordinaciones: []
    }
  },
  watch: {
    $route: {
      handler: function(route) {
        this.id = route.params.id && route.params.id
      },
      immediate: true
    }
  },
  async created() {
    await this.getRoles()
    await this.fetchData()
    await this.fetchDataListas()
  },
  methods: {
    fetchDataListas() {
      this.pageLoading = true
      getListaOne('COORD').then((response) => {
        response.data.enable = response.data.enable === 1
        // this.user = response.data
        this.pageLoading = false
        this.coordinaciones = [...response.data]
      })
    },

    async getRoles() {
      this.listLoading = true
      const params = {
        limit: this.numberItems,
        page: this.currentPage
      }
      await getAll(params).then(response => {
        this.roles = response.data.map(function(x) {
          x.enable = (x.enable === 1)
          return x
        })
        this.listLoading = false
      })
    },
    async fetchData() {
      this.pageLoading = true
      await getOne(this.id).then(response => {
        response.data.enable = (response.data.enable === 1)
        // this.user = response.data
        this.pageLoading = false
        this.userEditForm = { ...response.data }
      })
    },
    onSubmit() {
      const params = { ...this.userEditForm }
      params.enable = (params.enable) ? 1 : 0

      update(params).then(response => {
        this.$message(
          {
            showClose: true,
            message: 'Created Succefully!',
            type: 'success'
          }
        )
        this.$router.push({ path: '/user' })
      })
    },
    onCancel() {
      this.$router.push({ path: '/user' })
    }
  }
}
</script>
<style scoped>
img.row_avatar{
  width: 35px;
  border-radius: 50%;
}
</style>
