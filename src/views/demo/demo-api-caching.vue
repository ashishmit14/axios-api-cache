<template>
  <div>
    <h2>Set the Age for the API- Cache</h2>
    <h5>
      Default Age in Seconds:
      <input v-model="cacheMaxAge" type="number">
    </h5>

    <h2>Select a URL to test the API- Cache</h2>
    <h5>
      First Time Interval in Seconds:
      <input v-model="firstTimeInterVal" type="number">
    </h5>
    <h5>
      Second Time Interval in Seconds :
      <input v-model="secondTimeInterVal" type="number">
    </h5>

    <div v-if="urlMethodList && urlMethodList.length">
      <select @change="getSelectedurlMethodList(selectedMethod)" v-model="selectedMethod">
        <option selected disabled value>Select Request Type</option>
        <option
          v-for="item in urlMethodList"
          v-bind:key="item"
          v-bind:value="item"
        >{{item.toUpperCase()}}</option>
      </select>
      <button @click="reset">Reset</button>
    </div>
    <div v-if="urlDispList && urlDispList.length">
      <select @change="getSelectedUrl(selectedItem)" v-model="selectedItem">
        <option selected disabled value="null">{{('Select '+selectedMethod+' URL').toUpperCase()}}</option>
        <option v-for="item in urlDispList" v-bind:key="item.url" v-bind:value="item">{{item.url}}</option>
      </select>
    </div>
    <div v-if="msg && msg.length">
      <div v-for="item in msg">
        <p>{{item.dispMsg1}}</p>
        <p>{{item.dispMsg2}}</p>
        <p>
          <span v-for="xx in item.dispMsg2">_</span>
        </p>
      </div>
    </div>

    <div id="footer" v-if="msg && msg.length">CHECK CONSOLE FOR DETAILS</div>
  </div>
</template>

<script>
import axios from "axios";
import { setupCache } from "axios-cache-adapter";
import moment from "moment";

export default {
  data() {
    return {
      cache: null,
      api: null,
      cacheMaxAge: 3,
      mainResponse: null,
      secondaryResponse: null,
      thirdResponse: null,
      responseArr: [],
      urlMethodList: ["get", "post", "put"],
      urlDispList: [],
      urlList: [
        {
          url: "http://dummy.restapiexample.com/api/v1/employees",
          method: "get"
        },
        {
          url: "http://dummy.restapiexample.com/api/v1/employee/fff1",
          method: "get"
        },
        {
          url: "http://dummy.restapiexample.com/api/v1/create",
          method: "post"
        },
        {
          url: "http://dummy.restapiexample.com/api/v1/update/21",
          method: "put"
        }
      ],
      selectedUrl: "",
      selectedMethod: "",
      selectedItem: null,
      msg: [],
      setTimeOutObj1: null,
      setTimeOutObj2: null,
      firstTimeInterVal: 2,
      secondTimeInterVal: 7
    };
  },
  created() {
    console.log("Document created");
    this.cache = setupCache({
       maxAge: this.cacheMaxAge * 1000,
      // // Deactivate `clearOnStale` option so that we can actually read stale cache data
      // clearOnStale: false,
      // // Attempt reading stale cache data when response status is either 4xx or 5xx
      // readOnError: (error, request) => {
      //   return error.response.status >= 400 && error.response.status < 600;
      // },
      // // Tell adapter to attempt using response headers
      // readHeaders: true,
      // // For this example to work we disable query exclusion
      // exclude: { query: false }
    });
    this.api = axios.create({
      adapter: this.cache.adapter
    });
  },
  methods: {
    printTime(dt, msg) {
      console.log(
        msg,
        dt.hour() +
          ":" +
          dt.minute() +
          ":" +
          dt.second() +
          ":" +
          dt.millisecond()
      );
    },
    getRequest(numStr, url, methd) {
      var numSuffixStr = this.ordinal_suffix_of(numStr);
      var dt1 = new moment();

      this.printTime(dt1, "Before Request " + numStr + " : ");
      console.log("Cache store length:", length);

      this.api({
        url: url,
        method: methd
      }).then(async response => { 
        var dt2 = new moment();
        this.printTime(dt2, "On Response Recieved " + numStr + " : ");
        console.log("Request response:", response);
        var tempObj = {};
        tempObj.dispMsg1 = numSuffixStr + " RESPONSE RECIEVED";
        tempObj.dispMsg2 =
          "TOTAL TIME TAKEN" +
          " : " +
          moment(dt2).diff(dt1, "millisecond") +
          " ms";
        this.msg.push(tempObj);
        console.log(tempObj.dispMsg2);
        this.responseArr.push(response);
        //Interacting with the store, see `localForage` API.
        const length = await this.cache.store.length();
        console.log("Cache store:", this.cache);
        console.log("Cache store length:", length);
        var dt3 = new moment();
        this.printTime(dt3, "After Response Processed " + numStr + " : ");
      });
    },
    getSelectedurlMethodList(item) {
      this.selectedMethod = item;
      this.urlDispList = this.urlList.filter(o => {
        return o.method === item;
      });
    },
    getSelectedUrl(item) {
      if (item && item.method === "get") {
        this.selectedUrl = item.url;
        this.responseArr = [];
        this.msg = [];
        //request an API call
        this.getRequest("1", this.selectedUrl, this.selectedMethod);
        //request same API call
        this.setTimeOutObj1 = setTimeout(
          () => this.getRequest("2", this.selectedUrl, this.selectedMethod),
          2000
        );
        console.log("this.setTimeOutObj>>>>>>>>>>", this.setTimeOutObj);
        //request same API call
        this.setTimeOutObj2 = setTimeout(
          () => this.getRequest("3", this.selectedUrl, this.selectedMethod),
          7000
        );
      }
      if (item && item.method === "post") {
        this.selectedUrl = item.url;
        this.responseArr = [];
        this.msg = [];
        //request an API call
        this.getRequest("1", this.selectedUrl, this.selectedMethod);
        //request same API call
        this.setTimeOutObj1 = setTimeout(
          () => this.getRequest("2", this.selectedUrl, this.selectedMethod),
          this.firstTimeInterVal * 1000
        );
        console.log("this.setTimeOutObj>>>>>>>>>>", this.setTimeOutObj);
        //request same API call
        this.setTimeOutObj2 = setTimeout(
          () => this.getRequest("3", this.selectedUrl, this.selectedMethod),
          this.secondTimeInterVal * 1000
        );
      }
    },
    reset() {
      this.responseArr = null;
      this.selectedUrl = "";
      this.selectedMethod = "";
      this.selectedItem = null;
      this.urlDispList = [];
      this.responseArr = [];
      this.msg = [];
      clearTimeout(this.setTimeOutObj1);
      clearTimeout(this.setTimeOutObj2);
      this.firstTimeInterVal = 2;
      this.secondTimeInterVal = 7;
    },
    ordinal_suffix_of(i) {
      var j = i % 10,
        k = i % 100;
      if (j == 1 && k != 11) {
        return i + "st";
      }
      if (j == 2 && k != 12) {
        return i + "nd";
      }
      if (j == 3 && k != 13) {
        return i + "rd";
      }
      return i + "th";
    }
  }
};
</script>

<style scoped>
#footer {
  clear: both;
  position: relative;
  height: 200px;
  margin-top: -200px;
}
</style>