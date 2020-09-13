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
        <div id="information" v-if="curHumanIndex!=null && !displayRelation">
          <el-collapse accordion>
            <el-collapse-item title="姓名 Name" name="1">
              <div>
                <font face="微软雅黑">{{ curHuman.xm }}</font>
              </div>
            </el-collapse-item>
            <el-collapse-item title="户籍 Domicile" name="2">
              <div>
                <font face="微软雅黑">{{ curHuman.hj }}</font>
              </div>
            </el-collapse-item>
            <el-collapse-item title="所在地 Area" name="3">
              <div>
                <font face="微软雅黑">{{ curHuman.di }}</font>
              </div>
            </el-collapse-item>
            <el-collapse-item title="单位 Unit" name="4">
              <div>
                <font face="微软雅黑">{{ curHuman.wo }}</font>
              </div>
            </el-collapse-item>
            <el-collapse-item title="大学 University" name="5">
              <div>
                <font face="微软雅黑">{{ curHuman.sc }}</font>
              </div>
            </el-collapse-item>
            <el-collapse-item title="中学 High school," name="6">
              <div>
                <font face="微软雅黑">{{ curHuman.scc }}</font>
              </div>
            </el-collapse-item>
            <el-collapse-item title="小学 Primary school" name="7">
              <div>
                <font face="微软雅黑">{{ curHuman.sca }}</font>
              </div>
            </el-collapse-item>
            <el-collapse-item title="兴趣 Interest" name="8">
              <div>
                <font face="微软雅黑">{{ curHuman.xq }}</font>
              </div>
            </el-collapse-item>
            <el-collapse-item title="所在群组 Group" name="9">
              <div>
                <font face="微软雅黑">{{ curHuman.qun }}</font>
              </div>
            </el-collapse-item>
          </el-collapse>
          <br />
          <br />
          <br />
          <br />
          <el-button type="primary" plain @click="showRelatedHuman">可能认识的人</el-button>
        </div>
        <div v-if="displayRelation">
          <div class="related-human" v-for="(human,key) in relatedHuman" :key="key">
            <!-- 样例 -->
            <div>
              {{ human.name }}
              <br />
            </div>
            <div v-if="human.commonFriend.length">{{ human.commonFriend.length +"个共同好友"}}</div>
            <div v-if="human.sameGroup.length">
              {{ human.sameGroup.length + "个共同群："}}
              <div v-for="(group,index) in human.sameGroup" :key="index">{{group}}</div>
            </div>
            <div v-if="human.sameCompany">相同的工作单位</div>
          </div>
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
        .replace('"', "  ")
        .replace("[", " ")
        .replace("]", "");
      return curHuman;
    },
    relation: function () {
      if (this.curHumanIndex == null) {
        return [];
      }
      let relation = new Array(this.vertexes.length);

      for (let i = 0; i < relation.length; i++) {
        relation[i] = {
          book: true,
          commonFriend: [],
          sameCompany: false,
          sameUniv: false,
          sameMiddleSchool: false,
          samePrimarySchool: false,
          sameCity: false,
          sameHometown: false,
          sameHobby: [],
          sameGroup: [],
        };
      }

      let x = this.curHumanIndex;

      this.graph.adjacencyList[x].forEach((e) => {
        relation[e.from].book = false;
        relation[e.to].book = false;

        let friend = e.from == x ? e.to : e.from;
        this.graph.adjacencyList[friend].forEach((edge) => {
          relation[edge.from].commonFriend.push(this.graph.vertexes[friend].xm);
          relation[edge.to].commonFriend.push(this.graph.vertexes[friend].xm);
        });
      });

      let u = this.graph.vertexes[x];

      this.graph.vertexes.forEach((element, index) => {
        if (!relation[index].book) {
          return;
        }
        if (element.wo === u.wo) {
          relation[index].sameCompany = true;
        }

        if (element.sc === u.sc) {
          relation[index].sameUniv = true;
        }

        if (element.scc === u.scc) {
          relation[index].sameMiddleSchool = true;
        }

        if (element.sca === u.sca) {
          relation[index].samePrimarySchool = true;
        }

        if (element.hj === u.hj) {
          relation[index].sameHometown = true;
        }

        if (element.di === u.di) {
          relation[index].sameCity = true;
        }

        element.xq.forEach((hobby) => {
          if (u.xq.indexOf(hobby) !== -1) {
            relation[index].sameHobby.push(hobby);
          }
        });

        element.qun.forEach((group) => {
          if (u.qun.indexOf(group) !== -1) {
            relation[index].sameGroup.push(group);
          }
        });
      });

      return relation;
    },
    relatedHuman: function () {
      let res = [];
      this.relation.forEach((element, index) => {
        if (element.book) {
          let tmp = Object.assign({}, this.relation[index]);
          tmp.index = index;
          tmp.name = this.graph.vertexes[index].xm;
          tmp.score = this.calRelation(element);
          res.push(tmp);
        }
      });
      res.sort(function (a, b) {
        return b.score - a.score;
      });
      if (res.length > 5) {
        res = res.slice(0, 5);
      }
      return res;
    },
    vertexesDisp: function () {
      const activeVertexColor = "#0000CD";
      const normalVertexColor = "floralwhite";
      const lockVertexColor = "#fffaf030";
      const dispVertexColor = "gold";

      let vertexesDisp = [];
      //console.log(this.vertexes.length);
      for (let i = 0; i < this.vertexes.length; i++) {
        let element = this.vertexes[i];
        let t = {
          left: element.x + "px",
          top: element.y + "px",
          color: element.isActive
            ? activeVertexColor
            : element.isDisp
            ? dispVertexColor
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
          isDisp: false,
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
      this.displayRelation = false;
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
    calRelation(relation) {
      let res = 0;
      res += relation.commonFriend.length * 100;
      res += relation.sameGroup.length * 40;
      res += relation.sameHobby.length * 15;
      res += relation.sameCompany ? 20 : 0;
      res += relation.sameUniv ? 15 : 0;
      res += relation.sameMiddleSchool ? 12 : 0;
      res += relation.samePrimarySchool ? 10 : 0;
      res += relation.sameCity ? 5 : 0;
      res += relation.sameHometown ? 3 : 0;
      return res;
    },
    showRelatedHuman() {
      this.displayRelation = true;
      this.relatedHuman.forEach((element) => {
        console.log(element.index, this.vertexes[element.index]);
        this.vertexes[element.index].isDisp = true;
      });
    },
  },
  data: function () {
    return {
      vertexes: [],
      edges: [],
      curHumanIndex: null,
      displayRelation: false,
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

.related-human {
  border: 2px aliceblue solid;
}
</style>