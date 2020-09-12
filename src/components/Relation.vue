<template>
  <div class="relation">
    <el-container>
      <el-main>
        <div>
          <div
            class="edge"
            v-for="(edge,index) in edgesDisp"
            :key="index"
            v-bind:style="{
                        width: edge.width,
                        transform: edge.transform,
                        top: edge.top,
                        left: edge.left,
                        borderTopColor: edge.color
                    }"
          ></div>
        </div>
        <el-tooltip
          v-for="(vertex,index) in vertexesDisp"
          :key="index"
          placement="top"
          style="cursor:pointer"
        >
          <div slot="content">
            姓名：{{vertex.xm}}
            <br />
            单位：{{vertex.wo}}
          </div>
          <el-button
            class="human"
            icon="el-icon-user-solid"
            circle
            v-bind:style="{left:vertex.left,top:vertex.top,color:vertex.color,borderColor:vertex.color}"
            @click="setCurHuman(index)"
          ></el-button>
        </el-tooltip>
      </el-main>
      <el-aside style="width: 500px">
        <div id="information" v-if="curHumanIndex!=null">
          姓名:{{ curHuman.xm }}
          <br />
          户籍:{{ curHuman.hj }}
          <br />
          所在地:{{ curHuman.di }}
          <br />
          单位:{{ curHuman.wo }}
          <br />
          大学:{{ curHuman.sc }}
          <br />
          中学:{{ curHuman.scc }}
          <br />
          小学:{{ curHuman.sca }}
          <br />
          兴趣:{{ curHuman.xq }}
          <br />
          所在群组:{{ curHuman.qun }}
          <br />
          <el-button type="primary" plain>可能认识的人</el-button>
        </div>
      </el-aside>
    </el-container>
  </div>
</template>

<script>
export default {
  name: "Relation",
  props: {
    graph: Object,
  },
  computed: {
    curHuman: function () {
      if (this.curHumanIndex == null) {
        return {};
      }
      let curHuman = this.vertexes[this.curHumanIndex];
      curHuman.xq = (curHuman.xq + "")
        .replace('"', " ")
        .replace("[", " ")
        .replace("]", "");
      curHuman.qun = (curHuman.qun + "")
        .replace('"', " ")
        .replace("[", " ")
        .replace("]", "");
      return curHuman;
    },
    relatedHuman: function () {
      if (this.curHumanIndex == null) {
        return [];
      }
      let relation = new Array(this.vertexes.length);

      for (let i = 0; i < relation.length; i++) {
        relation[i] = {
          book: true,
          commonFriend: 0,
        };
      }

      for (
        let i = 0;
        i < this.graph.adjacencyList[this.curHumanIndex].length;
        i++
      ) {
        let e = this.this.graph.adjacencyList[this.curHumanIndex][i];
        relation[e.from].book = false;
        relation[e.to].book = false;

        let friend = e.from == this.curHumanIndex ? e.to : e.from;
        for (let j = 0; j < this.graph.adjacencyList[friend].length; j++) {
          relation[e.from].commonFriend++;
          relation[e.to].commonFriend++;
        }
        // 未完待续
      }
      return [];
    },
    vertexesDisp: function () {
      const activeVertexColor = "yellow";
      const normalVertexColor = "floralwhite";
      const lockVertexColor = "#fffaf030";

      let vertexesDisp = [];
      //console.log(this.vertexes.length);
      for (let i = 0; i < this.vertexes.length; i++) {
        let element = this.vertexes[i];
        let t = {
          left: element.x + "px",
          top: element.y + "px",
          color: element.isActive
            ? activeVertexColor
            : element.isLocked
            ? lockVertexColor
            : normalVertexColor,
        };
        for (let key in element) {
          t[key] = element[key];
        }
        vertexesDisp.push(t);
      }
      return vertexesDisp;
    },
    edgesDisp: function () {
      let edgesDisp = [];
      this.edges.forEach((edge) => {
        let u = this.vertexes[edge.from];
        let v = this.vertexes[edge.to];

        let dist = Math.sqrt(
          (u.x - v.x) * (u.x - v.x) + (u.y - v.y) * (u.y - v.y)
        );
        let angle = Math.atan2(v.y - u.y, v.x - u.x);
        const selectedEdgeColor = "#000";
        const activeEdgeColor = "floralwhite";
        const normalEdgeColor = "#fffaf060";
        edgesDisp.push({
          width: dist + "px",
          transform: "rotate(" + angle + "rad)",
          left: u.x + "px",
          top: u.y + "px",
          color: edge.isSelected
            ? selectedEdgeColor
            : edge.isActive
            ? activeEdgeColor
            : normalEdgeColor,
        });
      });
      return edgesDisp;
    },
  },
  created() {
    this.reset();
  },
  methods: {
    reset() {
      this.vertexes = [];
      this.edges = [];
      this.graph.vertexes.forEach((element) => {
        let t = {
          isActive: false,
          isLocked: false,
        };
        for (let key in element) {
          t[key] = element[key];
        }
        this.vertexes.push(t);
      });
      this.graph.edges.forEach((element) => {
        this.edges.push({
          from: element.from,
          to: element.to,
          dist: element.dist,
          index: element.index,
          isActive: false,
          isSelected: false,
        });
      });
      this.curHumanIndex = null;
    },
    setCurHuman(index) {
      this.reset();
      this.curHumanIndex = index;
      let curAdj = this.graph.adjacencyList[index];
      this.vertexes[index].isActive = true;
      let visited = new Array(this.graph.vertexes.length);
      for (let i = 0; i < curAdj.length; i++) {
        this.edges[curAdj[i].index].isActive = true;
        visited[this.edges[curAdj[i].index].to] = true;
        visited[this.edges[curAdj[i].index].from] = true;
      }
      for (let i = 0; i < this.graph.vertexes.length; i++) {
        if (i !== index && !visited[i]) {
          this.vertexes[i].isLocked = true;
        }
      }
    },
  },
  data: function () {
    return {
      vertexes: [],
      edges: [],
      curHumanIndex: null,
    };
  },
};
</script>

<style scoped>
.relation {
  height: 100%;
}

.el-container {
  height: 100%;
}

.el-main {
  background: #768b9c;
  width: 100%;
  height: 100%;
}

.human {
  position: absolute;
  background: #768b9c;
  border: 2px floralwhite solid;
  color: floralwhite;
  transform: translateX(-50%) translateY(-50%);
}

.edge {
  position: absolute;
  transform-origin: top left;
  border-top: 3px solid;
  transition: color 500ms cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

#information {
  position: relative;
  margin-top: 20%;
  margin-left: 20%;
  margin-right: 20%;
}
</style>