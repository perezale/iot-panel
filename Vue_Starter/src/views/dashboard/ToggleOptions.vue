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
      client: '',
      relays: [1, 2, 3, 4]
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
      _self.queryStatus('GET', 1)
      _self.queryStatus('GET', 2)
      _self.queryStatus('GET', 3)
      _self.queryStatus('GET', 4)
    })

    // client.on('message', this.handleMessage)
    client.on('message', _self.handleMessage)

    client.connect()

    this.init()
  },
  methods: {
    init: function () {
      let _self = this
      for (let relay of this.relays) {
        let relayInput = document.getElementById('relay-' + relay)
        console.log(relayInput)
        relayInput.addEventListener('change', function () {
          let checked = (relayInput.checked) ? '1' : '0'
          _self.queryStatus('POST', relay, checked)
        })
      }
    },
    handleMessage: function (topic, payload, details) {
      if (topic) {
        console.log('New message at: ' + topic)
        if (payload) {
          let msg = JSON.parse(payload)
          if (msg.module === 'relay') {
            let relay = msg.data.port
            let relayInput = document.getElementById('relay-' + relay)
            if (msg.data.value === '1') {
              relayInput.checked = true
            } else {
              relayInput.checked = false
            }
          }
        }
      }
    },
    queryStatus: function (op, port, value = 'NA') {
      let client = this.client
      let msg = {
        topic: 'esp-c0794b/inTopic',
        qos: 0,
        retain: true,
        payload: '{"module":"relay","operation":"' + op + '","data":{"port" : ' + port + ', "value": "' + value + '"}}'
      }
      client.publish(msg.topic, msg.payload)
    }
  }
}
</script>

<style lang="css">

</style>
