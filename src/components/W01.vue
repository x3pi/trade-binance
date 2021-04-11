<template>
  <div>
    <div class="row">
      <div class="col-4">
        <div class="flex">
          <q-table
            title="Bán"
            :rows="sells"
            :columns="columns"
            row-key="name"
            hide-bottom
          />
        </div>
      </div>
      <div class="col-4"></div>
      <div class="col-4">
        <div class="flex">
          <q-table
            title="Mua"
            :rows="buys"
            :columns="columns"
            row-key="name"
            hide-bottom
          />
        </div>
      </div>
    </div>
    <div>
      <h4>Mua: {{ buy }}</h4>
      <h4>Ban: {{ sell }}</h4>
    </div>
  </div>
</template>

<script>
export default {
  name: "W01",
  data() {
    return {
      buy: 0,
      sell: 0,
      buys: [],
      sells: [],
      columns: [
        {
          name: "Price",
          align: "center",
          label: "Price",
          field: "price",
          sortable: true,
          sort: (a, b) => {
            if (parseFloat(a) < parseFloat(b)) {
              return -1;
            }
            if (parseFloat(a) > parseFloat(b)) {
              return 1;
            }
            return 0;
          },
        },
        {
          name: "Quantity",
          label: "Quantity",
          field: "quantity",
        },
      ],
      rows: [
        {
          price: 87,
          quantity: 11,
        },
      ],
    };
  },
  methods: {
    init: function () {
      var exampleSocket = new WebSocket("wss://stream.binance.com/stream");
      exampleSocket.onopen = () => {
        exampleSocket.send(
          JSON.stringify({
            method: "SUBSCRIBE",
            params: [
              "!miniTicker@arr@3000ms",
              "rvnbnb@aggTrade",
              "rvnbnb@depth",
              "rvnbnb@kline_1d",
            ],
            id: 1,
          })
        );
      };
      var that = this;
      exampleSocket.onmessage = function (event) {
        var data = JSON.parse(event.data).data;
        if (data && data.e && data.e == "depthUpdate") {
          if (data.b && data.b[0]) {
            if (that.buys.length == 0) {
              that.buys.push({
                price: parseFloat(data.b[0][0]),
                quantity: parseFloat(data.b[0][1]),
              });
            } else {
              var index = -1;
              var element = false;
              that.buys.forEach(function (item, i) {
                if (item.price == parseFloat(data.b[0][0])) {
                  if (parseFloat(data.b[0][1]) == 0) {
                    that.buys.splice(i, 1);
                  } else {
                    index = i;
                    element = {
                      price: parseFloat(data.b[0][0]),
                      quantity: parseFloat(data.b[0][1]),
                    };
                  }
                  return;
                }
              });
              if (index != -1) {
                that.buys[index] = element;
              } else {
                if (parseFloat(data.b[0][1]) != 0) {
                  that.buys.push({
                    price: parseFloat(data.b[0][0]),
                    quantity: parseFloat(data.b[0][1]),
                  });
                }
              }
            }
            that.buy = data.b[0][0] + " : " + data.b[0][1];
          }
          if (data.a && data.a[0]) {
            if (that.sells.length == 0) {
              that.sells.push({
                price: parseFloat(data.a[0][0]),
                quantity: parseFloat(data.a[0][1]),
              });
            } else {
              var index = -1;
              var element = false;
              that.sells.forEach(function (item, i) {
                if (item.price == parseFloat(data.a[0][0])) {
                  if (parseFloat(data.a[0][1]) == 0) {
                    that.sells.splice(i, 1);
                    element = true;
                  } else {
                    index = i;
                    element = {
                      price: parseFloat(data.a[0][0]),
                      quantity: parseFloat(data.a[0][1]),
                    };
                  }
                  return;
                }
              });
              if (index != -1) {
                that.sells[index] = element;
              } else {
                if (parseFloat(data.a[0][1]) != 0) {
                  that.sells.push({
                    price: parseFloat(data.a[0][0]),
                    quantity: parseFloat(data.a[0][1]),
                  });
                }
              }
            }
            that.sell = data.a[0][0] + " : " + data.a[0][1];
          }
        }
      };
    },
  },
  mounted() {
    this.init();
  },
};
</script>

<style scoped></style>
