<template>
  <div class="load">
    <h1 :style="msgstyle">{{ vmmsg }}</h1>
    <div class="paras">
      <div class="para1" :style="genmapclass" @click.self="falseMapSource">New IO Map</div>
      <div class="para2" :style="hasmapclass" @click.self="trueMapSource">Use my own IO Map</div>
      <div class="para3"> </div>
    </div>
    <div class="paras">
      <div class="para1"><label for="posturl">Post URL:</label></div>
      <div class="para2"><input id="posturl" type="text" v-model="posturl"></div>
      <div class="para3"> </div>
    </div>
    <div class="paras" v-if="!hasIoMap">
      <div class="para1">
        <label for="outscheme">Your output scheme: </label>
      </div>
      <div class="para2">
        <select id="outscheme" @change="outColSetSelected" v-model="outColSet">
          <option value=""></option>
          <option v-for="(aset,asetid) in outColSets" :key="asetid" :value="aset">{{aset.name}}</option>
        </select>
      </div>
      <div class="para3"> </div>
    </div>

    <div class="paras">
      <div class="para1">
        <label for="inputfile">Input file: </label>
      </div>
      <div class="para2">
        <input id="inputfile" type="file" @change="genInColSet">
      </div>
      <div class="para3"> </div>
    </div>
    <div class="paras" v-if="hasIoMap">
      <div class="para1">
        <label for="mapfile">Your map file: </label>
      </div>
      <div class="para2">
        <input id="mapfile" type="file" @change="setIoMapFile">
      </div>
      <div class="para3">
        <input type="button" @click.prevent.stop="reloadIoMapFile" v-if="ioMapFile" value="Reload the map">
      </div>
    </div>
<!--      <input type="button" value="select iomap" @click.self="selectIoMap">-->
    <div class="paras">
      <div class="para1">
        <input type="button" value="Reset Map" v-on:click="resetMap">
      </div>
      <div class="para2">
        <input type="button" value="Help me map" v-on:click="helpMeMap" :disabled="helpmedisabled" :style="helpmestyle">
      </div>
      <div class="para3">
        <input type="button" value="Save map" v-on:click="saveIoMap">
      </div>
    </div>
    <div class="paras">
      <div class="para1">
        <input type="button" value="Load" v-on:click="loadFile">
      </div>
    </div>

        <div class="mapcontainer">
          <div class="incolset" v-if="inColSet">
            <div v-for="(inCol, inColId) in Object.values(inColSet)" :key="inColId">
              <div class="incolrow"  @click.prevent.stop="inColSelect(inCol)">
                <div style="border: 1px solid #42b983; flex: 0 0 100px" :style="inCol.style">{{inCol.name}}</div>
                <div style="border: 1px solid #42b983; flex: 0 0 20px" :style="inCol.style">{{inCol.colnum}}</div>
<!--                <div>{{inCol.style}}</div>-->
              </div>
            </div>
          </div>
          <div class="outcolset" v-if="outColSet">
            <div v-for="outCol in outColSet.cols" :key="outCol.name">
              <div class="incolrow">
<!--                <div>{{ outColView[outCol.name].selected ? "X" : " "}}</div>-->
                <div style="border: 0.5px solid #003383; flex: 0 0 50px;" @click.self="outColClear(outCol)">{{outCol.inCol ? "Clear": " "}}</div>
                <div style="border: 1px solid #003383; flex: 0 0 150px;" :style="outColView[outCol.name].style"
                     @click.self="outColSelect(outCol)">{{outCol.name}}</div>
                <div style="border: 1px solid #003383; flex: 0 0 120px;" :style="outColView[outCol.name].style"
                     @click.self="outColSelect(outCol)">{{outCol.inCol? outCol.inCol.name : " "}}</div>
                <!--              <div>{{outCol.path}}</div>-->
                <!--              <div>{{outCol.type}}</div>-->
              </div>
            </div>
          </div>
        </div>
    <div :class="loadProgressStyle"><p v-for="(line, iline) in loadProgress" :key="iline">{{line}}</p></div>
  </div>
</template>

<script>
  import Papa from "papaparse"
  import axios from "axios"
  import globals from "../assets/globals"
  import blob from "blob"
  import streamSaver from "streamsaver"
export default {
  name: 'Load',
  data: function() {
    return {
      loadProgress: [],
      loadProgressStyle: Object,
      inputFile: Object,
      totalRows: 0,
      loadedRows: 0,
      outColSets: Array,
      outColSet: Object,
      inColSet: Object,
      outColView: Object,
      ioMap: Object,
      ioMapFile: undefined,
      clickedOutcol: [Array, Object],
      clickedIncol: [Array, Object],
      hasIoMap: false,
      posturl: "http://192.168.168.112:8082/issues.xml?key=7091d25d404676ecbc770e16ee1e5262dede3d8e",
      msgstyle: Object,
      premsg:"",
    }
  },
  props: {
    msg: {
      type: String
    }
  },
  computed: {
    vmmsg: function() {
      return this.premsg || this.msg;
    },
    hasmapclass: function () {
      let vm = this
      return {
        backgroundColor: vm.hasIoMap ? '#8f8f8f': '#ffffff',
        border: '1px solid red'
      }
    },
    genmapclass: function () {
      let vm = this
      return {
        backgroundColor: vm.hasIoMap ? '#ffffff': '#8f8f8f',
        border: '1px solid red'
      }
    },
    helpmedisabled: function () {
      let vm = this
      if (vm.inColSet && Object.keys(vm.inColSet).length > 1 && vm.outColSet && vm.outColSet.cols) {
        return false
      } else {
        return true
      }
    },
    helpmestyle: function () {
      let vm = this
      if (vm.inColSet && Object.keys(vm.inColSet).length > 1 && vm.outColSet && vm.outColSet.cols) {
        return {
        }
      } else {
        return {
        }
      }
    }
  },
  watch: {
    inColSet: {
      handler: function() {
      },
      deep: true
    },
  },
  created() {
    this.outColSets = globals.outColSets
    if (this.outColSets.length == 1) {
      this.outColSet = this.outColSets[0]
      this.outColSetSelected()
    }
  },
  methods: {
    setInputFile (ev) {
      this.inputFile = ev.target.files[0];
    },
    outColSetSelected() {
      let vm = this
      if (!vm.outColSet || !vm.outColSet.cols || vm.outColSet.cols.length < 1) {
        vm.outColSet = null
        vm.premsg = "Cannot select outColSet. No cols find!!"
        vm.msgstyle = globals.errStyle
        return
      }
      vm.premsg = ""
      vm.msgstyle = ""
      vm.outColView = {}
      for (let i in vm.outColSet.cols) {
        let col = vm.outColSet.cols[i]
        this.outColView[col.name] = {}
      }
      vm.$forceUpdate()
    },
    genInColSet(ev) {
      let vm = this
      vm.setInputFile(ev)
      if (vm.hasIoMap) {
        return
      }
      const reader = new FileReader()
      reader.onload = (e) => {
        vm.inColSet = {}
        let x = e.target.result
        let parsed = Papa.parse(x)
        let rows = parsed.data
        let totalRows = rows.length
        for (let i in rows) {
          let row = rows[i]
          if (i < 10 && i < totalRows && (row.length > 2) && row[0].length > 0 && row[1].length > 0 && row[2].length > 0) {
            for (let col in row) {
              if(row[col].trim().length < 1) {
                break
              }
              vm.inColSet[row[col]]={name: row[col], colnum: col}
            }
            return;
          }
        }
      }
      reader.readAsText(this.inputFile)
    },
    loadFile () {
      const reader = new FileReader();
      let vm = this

      vm.loadProgress = []
      // reader.onload = e => this.$emit("load", e.target.result); //this line is commented since we do not want parent to process it.
      reader.onload = (e) => {
        let x = e.target.result
        let config = {
          quoteChar: ''
          // step: (row, parser) => {
          //   vm.loadProgress = vm.loadProgress + row;
          //   if (row.error()) {
          //     parser.stop();
          //   }
          // }
        }
        let parsed = Papa.parse(x, config).data
        vm.totalRows = parsed.length
        let idname = "", idcol = 0
        for (let k of Object.keys(vm.inColSet)) {
          idname = k
          idcol = vm.inColSet[k].colnum
          break
        }
        for (let irow in parsed) {
          if (parsed[irow][idcol] == idname) {
            console.log("======Header Row is skipped, row number " + irow.toString())
            continue
          }
          if (irow > 3) break
          vm.loadOne(parsed[irow])
        }
        vm.$forceUpdate()
      }
      reader.readAsText(this.inputFile)
    },
    loadOne(row) {
      let vm = this
      let anIssue = document.implementation.createDocument(null, vm.outColSet.rootpath)
      let issueRoot = anIssue.documentElement
      let ele0 = anIssue.createElement('project_id');
      issueRoot.appendChild(ele0)
      let txtNode = anIssue.createTextNode(1)
      ele0.appendChild(txtNode)

      ele0 = anIssue.createElement('tracker_id');
      txtNode = anIssue.createTextNode(3)
      ele0.appendChild(txtNode)
      issueRoot.appendChild(ele0)

      ele0 = anIssue.createElement('status_id');
      txtNode = anIssue.createTextNode(globals.issuestatuses.New)
      ele0.appendChild(txtNode)
      issueRoot.appendChild(ele0)

      ele0 = anIssue.createElement('priority_id');
      txtNode = anIssue.createTextNode(2)
      ele0.appendChild(txtNode)
      issueRoot.appendChild(ele0)

      ele0 = anIssue.createElement('assigned_to_id');
      txtNode = anIssue.createTextNode(5)
      ele0.appendChild(txtNode)
      issueRoot.appendChild(ele0)

      ele0 = anIssue.createElement('subject');
      txtNode = anIssue.createTextNode(row[1])
      ele0.appendChild(txtNode)
      issueRoot.appendChild(ele0)

      ele0 = anIssue.createElement('custom_fields')
      ele0.setAttribute('type', 'array')
      issueRoot.appendChild(ele0)
      let ele1 = anIssue.createElement('custom_field')
      // ele1.setAttribute('name', 'Epic AM/AC')
      ele1.setAttribute('id', 15)
      ele0.appendChild(ele1)
      ele0 = anIssue.createElement('value')
      ele1.appendChild(ele0)
      //6: Justin, 7: Will. Determined by owning application: row[vm.inColSet["Owning Application"].colnum], mapped by "Birdges/Cadence/Epicare Ambulatory/Grand Central/Kaleidoscope/Ophthain/Prelude/Referrals/Resolute professional bill"
      txtNode = anIssue.createTextNode('6')
      ele0.appendChild(txtNode)

      for (let outCol of vm.outColSet.cols) {
        if (outCol.inCol) {
          let inVal = row[outCol.inCol.colnum]
          let head = issueRoot
          let oPath = outCol.path || [{name: outCol.name}]
          for (let i in oPath) {
            let pathnode = oPath[i]
            let childNodes = globals.getChildEleByTag(head, pathnode.name)
            if (pathnode.multi) {
              ele0 = anIssue.createElement(pathnode.name)
              head.appendChild(ele0)
              head = ele0
              if (pathnode.attr) {
                for (let anAttr of pathnode.attr) {
                  let k = Object.keys(anAttr)[0]
                  if (k != 'name') head.setAttribute(k, anAttr[k])
                }
              }
            } else {
              if (childNodes.length > 1) {
                vm.premsg = "Abandoned processing due to error. outCol definition: " + JSON.stringify(outCol) + ". Reason: multi elements found for non multi element."
                vm.msgstyle = globals.errStyle
                return
              } else if (childNodes.length == 1) {
                if (pathnode.attr) {
                  for (let attr of pathnode.attr) {
                    let k = Object.keys(attr)[0]
                    if (k != "name" && childNodes[0].getAttribute(k) != attr[k]) {
                      vm.premsg = "Abandoned processing due to error. outCol definition: " + JSON.stringify(outCol) + ". Reason: Attribute inconsistent."
                      vm.msgstyle = globals.errStyle
                      return
                    }
                  }
                } else {
                  let names = childNodes[0].getAttributeNames()
                  if (names && names.length > 0) {
                    vm.premsg = "Abandoned processing due to error. outCol definition: " + JSON.stringify(outCol) + ". Reason: Attribute inconsistent."
                    vm.msgstyle = globals.errStyle
                    return
                  }
                }
                head = childNodes[0]
              } else {
                ele0 = anIssue.createElement(pathnode.name)
                head.appendChild(ele0)
                head = ele0
                if (pathnode.attr) {
                  for (let anAttr of pathnode.attr) {
                    let k = Object.keys(anAttr)[0]
                    if (k != "name") head.setAttribute(k, anAttr[k])
                  }
                }
              }
              if (i == oPath.length - 1) {
                txtNode = anIssue.createTextNode(inVal)
                head.appendChild(txtNode)
              }
            }
          }
        }
      }
      console.log("=====================: " + new XMLSerializer().serializeToString(anIssue))
      axios.post(vm.posturl,  new XMLSerializer().serializeToString(anIssue), {headers: {'Content-Type': 'application/xml'}}).then(function () {
        vm.loadedRows = vm.loadedRows + 1
        vm.loadProgress.push(row[1] + " is loaded. " + vm.loadedRows.toString() + "/" + vm.totalRows.toString() + "processed.")
      }).catch(function(e) {
        vm.premsg = JSON.stringify(e.response);
        vm.msgstyle = globals.errStyle
        vm.loadProgress.push("Abandoned. Error message shown above. Currently rows processed successfully or not is not logged in free version.")
      })
    },
    trueMapSource() {
      if (!this.hasIoMap) {
        this.outColSet = null
        this.inColSet = null
      }
      this.hasIoMap = true
    },
    falseMapSource() {
      if (this.hasIoMap) {
        this.outColSet = null
        this.inColSet = null
      }
      this.hasIoMap = false
    },
    saveIoMap() {
      let result = JSON.stringify({"incolset": this.inColSet, "outcolset": this.outColSet})
      let vm = this
      vm.saveToFile(result)
    },
    saveToFile(s) {
      let aBlob = new blob([s])
      let fileStream = streamSaver.createWriteStream("tobe iomap.txt")
      let readableStream = aBlob.stream()
      if (window.WritableStream && readableStream.pipeTo) {
        return readableStream.pipeTo(fileStream).then(() => {console.log("Writing tobe iomap.txt done")})
      }
      let writer = fileStream.getWriter()
      let reader = readableStream.getWriter()
      let pump = () =>
        reader.read().then(res =>
          res.done? writer.close() : writer.write(res.value).then(pump)
        )
      pump()
    },
    selectIoMap() {
      this.premsg = 'IO Map deserialization is not impleemnt yet!'
      this.msgstyle=globals.errStyle
    },
    setIoMapFile(ev) {
      let vm = this
      vm.ioMapFile = ev.target.files[0]
      vm.reloadIoMapFile()
    },
    reloadIoMapFile() {
      let vm = this
      if (!vm.ioMapFile) return
      const reader = new FileReader()
      reader.onload = (e) => {
        let x = e.target.result
        let result = JSON.parse(x)
        // console.log("======result: " + JSON.stringify(result))
        vm.inColSet = result.incolset
        vm.outColSet = result.outcolset
        vm.outColSets = [vm.outColSet]
        vm.outColSetSelected()
        for (let outCol of vm.outColSet.cols) {
          if (outCol.inCol) {
            vm.inColSet[outCol.inCol.name].style = globals.styleOutColAssigned
          }
        }
        vm.$forceUpdate()
      }
      reader.readAsText(vm.ioMapFile)
    },
    inColSelect(incol) {
      let vm = this
      vm.clickedIncol = incol
      for (let v of Object.values(vm.inColSet)) {
        if (v.style != globals.styleOutColAssigned) {
          v.style = globals.styleOutColUnselected
        }
      }
      incol.style = globals.styleOutColSelected
      this.$forceUpdate()
    },
    outColSelect(outCol) {
      let vm = this
      if (!vm.clickedIncol) {
        vm.premsg = "Please select a input column first!!"
        vm.msgstyle = globals.errStyle
      } else {
        if (outCol.inCol && outCol.inCol.name != vm.clickedIncol.name) {
          outCol.inCol.style = globals.styleOutColUnselected
        }
        outCol.inCol = vm.clickedIncol
        vm.clickedIncol.style = globals.styleOutColAssigned
        vm.clickedIncol = null
      }
      vm.$forceUpdate()
    },
    outColClear(outCol) {
      if (outCol.inCol) {
        for (let anInCol of Object.values(this.inColSet)) {
          anInCol.style = globals.styleOutColUnselected
        }
        outCol.inCol = null
        for (let anOutCol of this.outColSet.cols) {
          if (anOutCol.inCol) {
            this.inColSet[anOutCol.inCol.name].style = globals.styleOutColAssigned
          }
        }
        this.$forceUpdate()
      }
    },
    helpMeMap() {
      let vm = this
      if (vm.inColSet && Object.keys(vm.inColSet).length > 1 && vm.outColSet && vm.outColSet.cols){
        for (let inCol of Object.values(vm.inColSet)) {
          inCol.style = globals.styleOutColUnselected
        }
        for (let outCol of vm.outColSet.cols) {
          if (vm.inColSet[outCol.name]) {
            outCol.inCol = vm.inColSet[outCol.name]
            vm.inColSet[outCol.name].style = globals.styleOutColAssigned
          }
        }
        vm.$forceUpdate()
      }
    },
    resetMap() {
      let vm = this
      if (vm.outColSet && vm.outColSet.cols) {
        for (let outCol of vm.outColSet.cols) {
          outCol.inCol = null
        }
      }
      if (vm.inColSet) {
        for (let inCol of Object.values(vm.inColSet)) {
          inCol.style = null
        }
      }
      vm.$forceUpdate()
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.load {
  text-align: left;
  margin-left: 5%;
}
  .incolrow{
    display: flex;
    flex-wrap: nowrap;
    margin: 0%;
    align-content: center;
    /*align-items: stretch;*/
    /*justify-content: stretch;*/
  }
  .mapcontainer{
    clear: both;
  }
  .incolset {
    display: block;
    width: 35%;
    float:left;
  }
  .outcolset {
    display: block;
    float: left;
    margin-left: 1em;
  }
  .paras {
    display: flex;
    flex-wrap: wrap;
    margin: 0 0 auto;
    align-content: center;
    align-items: stretch;
    justify-content: stretch;
  }
  .para1 {
    flex-basis: 28%
  }
  .para2 {
    flex-basis: 40%
  }
  .para3 {
    flex-basis: 25%;
  }
</style>
