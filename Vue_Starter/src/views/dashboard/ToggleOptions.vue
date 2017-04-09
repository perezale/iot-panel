<template>
  <div class="animated fadeIn">
    <div class="row">
      <div class="col-md-12">
        <div class="card">
          <div class="card-header">
            Relay #1
            <label class="switch switch-3d switch-success float-right mb-0"><input id="relay-1" type="checkbox" class="switch-input"> <span data-on="On" data-off="Off" class="switch-label"></span> <span class="switch-handle"></span></label></div>
        </div>
        <div class="card">
          <div class="card-header">
            Relay #2
            <label class="switch switch-3d switch-success float-right mb-0"><input id="relay-2" type="checkbox" class="switch-input"> <span data-on="On" data-off="Off" class="switch-label"></span> <span class="switch-handle"></span></label></div>
        </div>
        <div class="card">
          <div class="card-header">
            Relay #3
            <label class="switch switch-3d switch-success float-right mb-0"><input id="relay-3" type="checkbox" class="switch-input"> <span data-on="On" data-off="Off" class="switch-label"></span> <span class="switch-handle"></span></label></div>
        </div>
        <div class="card">
          <div class="card-header">
            Relay #4
            <label class="switch switch-3d switch-success float-right mb-0"><input id="relay-4" type="checkbox" class="switch-input"> <span data-on="On" data-off="Off" class="switch-label"></span> <span class="switch-handle"></span></label></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'toggle-options',
  data () {
    return {
      client: ''
    }
  },
  mounted: function () {
    let _self = this
    let MqttClient = window.MqttClient

    let client = MqttClient({
      host: 'cuys-srv.cloudapp.net',
      port: 61614,
      username: 'guest',
      password: 'password'
    })
    this.client = client

    client.on('connecting', function () { console.log('Connecting...') })

    client.on('connect', function () {
      console.log('Connected!')
      _self.client.subscribe('esp-c0794b/outTopic', _self.handleMessage)
      _self.queryStatus(1)
      _self.queryStatus(2)
      _self.queryStatus(3)
      _self.queryStatus(4)
    })

    // client.on('message', this.handleMessage)
    client.on('message', _self.handleMessage)

    client.connect()
  },
  methods: {
    handleMessage: function (topic, payload, details) {
      if (topic) {
        console.log('New message at: ' + topic)
        if (payload) {
          let msg = JSON.parse(payload)
          console.log(payload)
          if (msg.module === 'relay') {
            let relay = msg.data.port
            var relayInput = document.getElementById('relay-' + relay)
            if (msg.data.value === '1') {
              relayInput.selected = true
            } else {
              relayInput.selected = false
            }
          }
        }
      }
    },
    queryStatus: function (port) {
      let client = this.client
      let msg = {
        topic: 'esp-c0794b/inTopic',
        qos: 0,
        retain: true,
        payload: '{"module":"relay","operation":"GET","data":{"port" : ' + port + ', "value": "0"}}'
      }
      client.publish(msg.topic, msg.payload)
    }
  }
}
</script>

<style lang="css">

</style>
