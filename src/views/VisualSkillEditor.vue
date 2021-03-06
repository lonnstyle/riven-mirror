<template>
  <div class="visual-skill-editor">
    <div class="op">
      <div class="title">操作</div>
      <el-button type="primary" size="small" @click="saveAll">全部导出</el-button>
      <el-button type="primary" size="small" style="margin-right:8px" @click="save">导出当前战甲</el-button>
      <router-link :to="`/warframe/${wfClass}`">
        <el-button size="small" style="margin-right:8px" @click="apply">试用</el-button>
      </router-link>
      <el-checkbox v-model="saveRaw">RAW格式</el-checkbox>
    </div>
    <div class="warframe-list">
      <div class="title">选择战甲</div>
      <el-radio-group v-model="wfClass" size="small">
        <el-radio class="wfid" :key="id" v-for="id in wfClasses" :label="id" border />
      </el-radio-group>
    </div>
    <div class="skill-list">
      <div class="skill-box" :key="skill.id" v-for="skill in showSkills">
        <VisualSkill class="skill" v-model="skills[skillNameMap[skill.id]]" />
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { cloneDeep, mapValues, uniq } from "lodash-es";
import { Vue, Component, Watch, Prop } from "vue-property-decorator";
import {
  AbilityProp,
  AbilityData,
  AbilityEnhance,
  AbilityFormData,
  AdvancedAbilityPropValue,
  AbilityType,
  WarframeProperty,
  AbilityPropTypes,
} from "@/warframe/codex";
import { _abilityData, _warframeData, registerAbilityData } from "@/warframe/codex/warframe.data";
import VisualSkill from "@/components/VisualSkill.vue";
import { i18n } from "@/i18n";

function convertSkills(src: string) {
  return src
    .replace(/\{"value":(-?[\d\.]+)\}/g, "$1")
    .replace(/\{"value":(-?[\d\.]+),"bind":\[\["t",(-?[\d\.]+)\]\]\}/g, "S($1,$2)")
    .replace(/\{"value":(-?[\d\.]+),"bind":\[\["u",(-?[\d\.]+)\]\]\}/g, "D($1,$2)")
    .replace(/\{"value":(-?[\d\.]+),"bind":\[\["x",(-?[\d\.]+)\]\]\}/g, "E($1,$2)")
    .replace(/\{"value":(-?[\d\.]+),"bind":\[\["g",(-?[\d\.]+)\]\]\}/g, "R($1,$2)")
    .replace(/"(\w+?)":/g, "$1:");
}
@Component({ components: { VisualSkill } })
export default class VisualSkillEditor extends Vue {
  skills: AbilityData[] = null;
  skillNameMap: { [name: string]: number } = null;
  wfClass = "Ash";
  saveRaw = false;
  get wfClasses() {
    return uniq(_warframeData.map(v => v.className || v.id));
  }
  get showSkills() {
    let wf = _warframeData.find(v => v.id === this.wfClass);
    let skills = this.skills.filter(v => wf.abilities.includes(v.id));
    // console.log(this.wfClass, skills)
    return skills;
  }
  apply() {
    registerAbilityData(this.skills);
  }
  save() {
    let wf = _warframeData.find(v => v.id === this.wfClass);
    let skills = this.skills.filter(v => wf.abilities.includes(v.id));
    console.log(skills);
    let data = JSON.stringify(skills);
    let blob = new Blob([this.saveRaw ? data : convertSkills(data)], { type: "application/json" });
    let a = document.createElement("a");
    a.download = this.wfClass.toLowerCase() + "-skills.json";
    a.href = URL.createObjectURL(blob);
    a.click();
  }
  saveAll() {
    let skills = this.skills;
    console.log(skills);
    let data = JSON.stringify(skills);
    let blob = new Blob([this.saveRaw ? data : convertSkills(data)], { type: "application/json" });
    let a = document.createElement("a");
    a.download = "skills.json";
    a.href = URL.createObjectURL(blob);
    a.click();
  }
  // === 生命周期钩子 ===
  beforeMount() {
    this.skills = _abilityData.map(v => {
      let a = cloneDeep(v);
      const mappv = (pv: any) => {
        if (Array.isArray(pv)) return pv.map(([vn, vv]) => (typeof vv === "number" ? [vn, { value: vv }] : [vn, vv]));
        return typeof pv === "number" ? { value: pv } : pv;
      };
      a.props = mapValues(a.props, prop => {
        return Array.isArray(prop) ? prop.map(prop => mapValues(prop, mappv)) : mapValues(prop, mappv);
      }) as any;
      return a;
    });
    this.skillNameMap = this.skills.reduce((a, b, i) => ((a[b.id] = i), a), {});
  }
}
</script>

<style lang="less" scoped>
.warframe-list {
  .title {
    font-size: 20px;
    margin: 8px 12px;
    font-weight: bold;
  }
  .el-radio-group .wfid {
    margin: 4px 2px;
    background: #fff;
  }
}
.skill-list {
  .skill-box {
    border: 1px solid #dcdfe6;
    margin-top: 14px;
    padding: 8px;
    border-radius: 4px;
    background: #fff;
    transition: border-color 0.4s;
    &:hover {
      border-color: #6199ff;
    }
  }
}
.op {
  .title {
    display: inline-block;
    padding: 0 12px;
  }
  background: #fff;
  padding: 8px;
  border-radius: 4px;
}
</style>
