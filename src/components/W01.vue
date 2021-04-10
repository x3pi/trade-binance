<template>
  <div >
    <h4>Ban: {{ buy }}</h4>
    <h4>Mua: {{ sell }}</h4>
  </div>
</template>

<script>

export default {
  name: 'W01',
  data() {
    return {
      buy: 0,
      sell: 0,
    }
  },
  methods: {
    init: function () {
      var exampleSocket = new WebSocket("wss://stream.binance.com/stream");
      exampleSocket.onopen = () => {
        console.log("open")
        exampleSocket.send(JSON.stringify({"method":"SUBSCRIBE","params":["!miniTicker@arr@3000ms","ethusdt@aggTrade","ethusdt@depth","ethusdt@kline_1d"],"id":1}));
      }
      var that = this
      exampleSocket.onmessage = function (event) {
        var data = JSON.parse(event.data).data
          if(data && data.e && data.e == "aggTrade"){
            if(data.m == true){
              if(that.buy == 0 ||that.buy < data.p){
                console.log(that.buy)
                that.buy = data.p
              }
            }else{
              if(that.sell == 0 || that.sell > data.p){
                that.sell = data.p
              }
            }
          }
      }
    }
  },
  mounted() {
    this.init()
  },
}
</script>

<style scoped>

</style>
