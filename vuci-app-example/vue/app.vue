<template>
  <div>
    <vuci-form class="mainForm" :uci-config="config" :key="formComponentKey">
      <vuci-typed-section type="rules" :columns="columns">
        <template #name="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="name" rules="host" />
        </template>
        <template #address="{ s }">
          <vuci-form-item-dummy
            :uci-section="s"
            name="address"
            rules="ipaddr"
          />
        </template>
        <template #netmask="{ s }">
          <vuci-form-item-dummy
            :uci-section="s"
            name="netmask"
            rules="ipaddr"
          />
        </template>
        <template #extra="{ s }">
          <a-button
            :uci-section="s"
            type="primary"
            name="edit"
            @click="editSection(s)"
            >Edit</a-button
          >
          <a-tooltip :title="$t('Delete')">
            <a-button
              :uci-section="s"
              type="danger"
              name="delete"
              @click="delSection(s)"
              >Delete</a-button
            >
          </a-tooltip>
        </template>
      </vuci-typed-section>
    </vuci-form>
    <a-form>
      <div class="flex">
        <a-input
          v-model="newInterfaceName"
          placeholder="Add new interface here.. "
        ></a-input>
        <a-button type="primary" @click="addInterface">Add</a-button>
      </div>
    </a-form>

    <a-modal v-model="modalVisible">
      <edit-modal
        :config="config"
        :modalComponentKey="modalComponentKey"
        :sectionToEdit="sectionToEdit"
        :protocolSelected="protocolSelected"
        v-on:toggleProtocol="toggleProtocol"
      >
      </edit-modal>
    </a-modal>
  </div>
</template>

<script>
import EditModal from './components/modal'
export default {
  components: {
    'edit-modal': EditModal
  },
  data () {
    return {
      sid: '',
      newInterfaceName: '',
      formComponentKey: 1,
      modalComponentKey: 1,
      config: 'custom',
      type: 'rules',
      protocolSelected: '',
      modalVisible: false,
      columns: [
        {
          name: 'name',
          label: this.$t('Interface Name'),
          width: 140,
          key: 'key'
        },
        { name: 'address', label: this.$t('Address'), width: 140 },
        { name: 'netmask', label: this.$t('Netmask'), width: 140 },
        { name: 'extra', width: 140, scopedSlots: { customRender: 'edit' } }
      ],
      sectionToEdit: ''
    }
  },
  methods: {
    toggleProtocol () {
      this.protocolSelected === 'DHCP' ? this.protocolSelected = 'static' : this.protocolSelected = 'DHCP'
    },
    editSection (s) {
      this.protocolSelected = s.protocol
      this.sectionToEdit = s['.name']
      this.modalVisible = true
      this.refreshComponents()
    },
    delSection (s) {
      this.$confirm({
        title: this.$t('Delete'),
        onOk: () => {
          return new Promise((resolve) => {
            this.$uci.del(this.config, s['.name'])
            resolve()
          })
            .then(() => this.$uci.save())
            .then(() => this.$uci.apply())
            .then(() => this.load(this.config))
        }
      })
    },

    refreshComponents () {
      this.modalComponentKey++
      this.formComponentKey++
    },
    async addInterface () {
      return new Promise((resolve) => {
        this.sid = this.$uci.add(this.config, this.type)
        resolve()
        this.$uci.set(this.config, this.sid, 'name', this.newInterfaceName)
        this.$uci.set(this.config, this.sid, 'protocol', this.protoOptions[0])
        this.$uci.set(this.config, this.sid, 'DHCP', 1)
      })
        .then(() => this.$uci.save())
        .then(() => {
          this.$uci.apply()
          this.newInterfaceName = ''
        })
        .then(() => this.load(this.config))
    },
    async load (config) {
      await this.$uci.load(config)
      this.refreshComponents()
    }
  }
}
</script>

<style>
.flex {
  display: flex;
  width: 30%;
  margin: 0 auto;
}

button {
  margin-left: 8px;
}

.mainForm .ant-form-item-children div {
  display: none;
}

.ant-modal-footer {
  display: none;
}
</style>
