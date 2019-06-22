<template>
  <div class="ex1">
    <h2>Set the Age for the API- Cache</h2>
    <h5>
      Default Age in Seconds:
      <input v-model="cacheMaxAge" type="number">
      <button class="button" @click="reset">Reset</button>
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

    <p v-if="urlMethodList && urlMethodList.length" style="width:300px; margin: auto; padding: 10px;">
      <select @change="getSelectedurlMethodList(selectedMethod)" v-model="selectedMethod" class="select-css">
        <option selected disabled value>Select Request Type</option>
        <option
          v-for="item in urlMethodList"
          v-bind:key="item"
          v-bind:value="item"
        >{{item.toUpperCase()}}</option>
      </select>
    </p>
    <p v-if="urlDispList && urlDispList.length" style="width:600px; margin: auto; padding: 10px;">
      <select @change="getSelectedUrl(selectedItem)" v-model="selectedItem" class="select-css">
        <option selected disabled value="null">{{('Select '+selectedMethod+' URL').toUpperCase()}}</option>
        <option v-for="item in urlDispList" v-bind:key="item.url" v-bind:value="item">{{item.url}}</option>
      </select>
    </p>
    <div v-if="msg && msg.length" id="datamsg">
      <div v-for="item in msg">
        <p style="font-weight:800">{{item.dispMsg1}}</p>
        <p style="font-weight:500">{{item.dispMsg2}}</p>
        <p>
          <span v-for="xx in item.dispMsg2">_</span>
        </p>
      </div>
    </div>

    <div id="footer" v-if="msg && msg.length">Check Console For Details</div>
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
      maxAge: this.cacheMaxAge * 1000
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
  mounted(){
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
        console.log("__________________________________________________________________________");
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
      this.cacheMaxAge = 3;
      console.clear();
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
  },
  updated(){
  }
};
</script>

<style scoped>
div.ex1 {
  background-color: lightblue;
  width: 100%;
  height: 651px;
  overflow: scroll;
}
input[type=number] {
  width: 10%;
  padding: 12px 20px;
  margin: 8px 0;
  display: inline-block;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}
#footer {
  position: fixed;
  left: 0px;
  bottom: 0px;
  height: 30px;
  width: 100%;
  font-weight: 900;
  color: rgb(0, 0, 0);
  background-color: rgb(113, 159, 189); /* For browsers that do not support gradients */
  background-image: linear-gradient(to bottom right, rgba(27, 13, 13, 0.041), rgba(174, 247, 146, 0.781));
}
.button {
  background-color: #4CAF50; /* Green */
  border: none;
  color: white;
  padding: 5px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 6px 6px;
  cursor: pointer;
  height: 40px;
}
/* class applies to select element itself, not a wrapper element */
.select-css {
	display: block;
	font-size: 16px;
	font-family: sans-serif;
	font-weight: 700;
	color: #444;
	line-height: 1.3;
	padding: .6em 1.4em .5em .8em;
	width: 100%;
	max-width: 100%; /* useful when width is set to anything other than 100% */
	box-sizing: border-box;
	margin: 0;
	border: 1px solid #aaa;
	box-shadow: 0 1px 0 1px rgba(0,0,0,.04);
	border-radius: .5em;
	-moz-appearance: none;
	-webkit-appearance: none;
	appearance: none;
	background-color: #fff;
	/* note: bg image below uses 2 urls. The first is an svg data uri for the arrow icon, and the second is the gradient. 
		for the icon, if you want to change the color, be sure to use `%23` instead of `#`, since it's a url. You can also swap in a different svg icon or an external image reference
		
	*/
	background-image: url('data:image/svg+xml;charset=US-ASCII,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%22292.4%22%20height%3D%22292.4%22%3E%3Cpath%20fill%3D%22%23007CB2%22%20d%3D%22M287%2069.4a17.6%2017.6%200%200%200-13-5.4H18.4c-5%200-9.3%201.8-12.9%205.4A17.6%2017.6%200%200%200%200%2082.2c0%205%201.8%209.3%205.4%2012.9l128%20127.9c3.6%203.6%207.8%205.4%2012.8%205.4s9.2-1.8%2012.8-5.4L287%2095c3.5-3.5%205.4-7.8%205.4-12.8%200-5-1.9-9.2-5.5-12.8z%22%2F%3E%3C%2Fsvg%3E'),
	  linear-gradient(to bottom, #ffffff 0%,#e5e5e5 100%);
	background-repeat: no-repeat, repeat;
	/* arrow icon position (1em from the right, 50% vertical) , then gradient position*/
	background-position: right .7em top 50%, 0 0;
	/* icon size, then gradient */
	background-size: .65em auto, 100%;
}
/* Hide arrow icon in IE browsers */
.select-css::-ms-expand {
	display: none;
}
/* Hover style */
.select-css:hover {
	border-color: #888;
}
/* Focus style */
.select-css:focus {
	border-color: #aaa;
	/* It'd be nice to use -webkit-focus-ring-color here but it doesn't work on box-shadow */
	box-shadow: 0 0 1px 3px rgba(59, 153, 252, .7);
	box-shadow: 0 0 0 3px -moz-mac-focusring;
	color: #222; 
	outline: none;
}

/* Set options to normal weight */
.select-css option {
	font-weight:normal;
}

/* Support for rtl text, explicit support for Arabic and Hebrew */
*[dir="rtl"] .select-css, :root:lang(ar) .select-css, :root:lang(iw) .select-css {
	background-position: left .7em top 50%, 0 0;
	padding: .6em .8em .5em 1.4em;
}

/* Disabled styles */
.select-css:disabled, .select-css[aria-disabled=true] {
	color: graytext;
	background-image: url('data:image/svg+xml;charset=US-ASCII,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%22292.4%22%20height%3D%22292.4%22%3E%3Cpath%20fill%3D%22graytext%22%20d%3D%22M287%2069.4a17.6%2017.6%200%200%200-13-5.4H18.4c-5%200-9.3%201.8-12.9%205.4A17.6%2017.6%200%200%200%200%2082.2c0%205%201.8%209.3%205.4%2012.9l128%20127.9c3.6%203.6%207.8%205.4%2012.8%205.4s9.2-1.8%2012.8-5.4L287%2095c3.5-3.5%205.4-7.8%205.4-12.8%200-5-1.9-9.2-5.5-12.8z%22%2F%3E%3C%2Fsvg%3E'),
	  linear-gradient(to bottom, #ffffff 0%,#e5e5e5 100%);
}

.select-css:disabled:hover, .select-css[aria-disabled=true] {
	border-color: #aaa;
}

</style>