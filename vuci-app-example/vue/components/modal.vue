<template>
    <vuci-form :uci-config="config" :key="modalComponentKey">
      <vuci-named-section :name="sectionToEdit" v-slot="{ s }">
        <vuci-form-item-select
          @change="protocolSelecting($event), onDHCP()"
          :uci-section="s"
          :label="'Protocol'"
          :options="protoOptions"
          name="protocol"
          placeholder="Select your protocol.."
        />
        <div v-if="protocolSelected !== 'DHCP'" :key="protocolSelected">
          <vuci-form-item-input
            :uci-section="s"
            :label="'Address'"
            name="address"
            placeholder="192.168.1.1."
            rules="ip4addr"
            required
          />
          <vuci-form-item-input
            :uci-section="s"
            :label="'Netmask'"
            name="netmask"
            placeholder="255.255.255.255"
            rules="netmask4"
            required
          />
          <vuci-form-item-input
            :uci-section="s"
            :label="'Gateway'"
            name="gateway"
            placeholder="192.168.1.1"
            rules="ip4addr"
          />
          <vuci-form-item-list :uci-section="s" :label="'DNS'" name="DNS" />
          <vuci-form-item-switch
            :uci-section="s"
            :label="'DHCP'"
            name="DHCP"
            true-value="1"
            false-value="0"
          />
        </div>
      </vuci-named-section>
    </vuci-form>

</template>

<script>
export default {
  name: 'EditModal',
  props: ['modalComponentKey', 'sectionToEdit', 'modalVisible', 'config', 'protocolSelected'],
  data () {
    return {
      protoOptions: ['static', 'DHCP'],
      protocolChanged: ''
    }
  },
  methods: {
    protocolSelecting (e) {
      return (this.protocolChanged = e.dependExprValue.replace(
        /^["'](.+(?=["']$))["']$/,
        '$1'
      ))
    },
    onDHCP () {
      console.log('onDHCP protocol select is : ', this.protocolSelected)
      this.$emit('toggleProtocol')
      if (this.protocolSelected === 'DHCP') {
        this.$uci.set(this.config, this.sectionToEdit, 'address', '')
        this.$uci.set(this.config, this.sectionToEdit, 'netmask', '')
        this.$uci.set(this.config, this.sectionToEdit, 'gateway', '')
        this.$uci.set(this.config, this.sectionToEdit, 'DNS', '')
        this.$uci.set(this.config, this.sectionToEdit, 'DHCP', '')
      }
      console.log('dhcp changed', this.protocolChanged)
    }
  }
}
</script>

<style scoped>
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
