<template>
  <div>
    <table class="tableBox" @mousedown="handleMouseDown">
      <div
        class="mask"
        v-show="is_show_mask"
        :style="
          'width:' +
          mask_width +
          'left:' +
          mask_left +
          'height:' +
          mask_height +
          'top:' +
          mask_top
        "
      ></div>
      <thead>
        <tr>
          <td
            v-for="(item, index) in columns"
            @click="(e) => tableHeadClick(e, index)"
            style="position: relative"
          >
            {{ item.title }}
            <!-- isShowMenu[index] -->
          </td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in data">
          <td
            v-for="(item2, index2) in columns"
            :class="{ active: index2 === curIndex }"
          >
            {{ item[item2.key] }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  props: ["columns", "data"],
  data() {
    return {
      menu_left: 0,
      menu_top: 0,
      isShowMenu: [],
      select_list: [],
      end_x: 0,
      end_y: 0,
      start_x: 0,
      start_y: 0,
      box_screen_top: 0,
      box_screen_left: 0,
      is_show_mask: false,
      selectedList: [],
      dataList: ["1", "2", "3"],
      curIndex: "",
    };
  },
  watch: {},
  computed: {
    mask_width() {
      return `${Math.abs(this.end_x - this.start_x)}px;`;
    },
    mask_height() {
      return `${Math.abs(this.end_y - this.start_y)}px;`;
    },

    mask_left() {
      return `${Math.min(this.start_x, this.end_x) - this.box_screen_left}px;`;
    },

    mask_top() {
      return `${Math.min(this.start_y, this.end_y) - this.box_screen_top}px;`;
    },
  },
  methods: {
    tableHeadClick(e, index) {
      //获取当前列
      this.curIndex = index;
      this.menu_left = event.clientX;
    },
    /* 方法 */
    handleMouseDown(event) {
      if (event.target.tagName === "SPAN") return false;

      this.is_show_mask = true;
      this.start_x = event.clientX;
      this.start_y = event.clientY;
      this.end_x = event.clientX;
      this.end_y = event.clientY;

      document.body.addEventListener("mousemove", this.handleMouseMove);
      document.body.addEventListener("mouseup", this.handleMouseUp);
    },
    handleMouseMove(event) {
      this.end_x = event.clientX;
      this.end_y = event.clientY;
    },
    handleMouseUp() {
      document.body.removeEventListener("mousemove", this.handleMouseMove);
      document.body.removeEventListener("mouseup", this.handleMouseUp);
      this.is_show_mask = false;
      this.handleDomSelect();
      this.resSetXY();
    },
    //处理选框与矩形区域是否相交
    collide(rect1, rect2) {
      const maxX = Math.max(rect1.x + rect1.width, rect2.x + rect2.width);
      const maxY = Math.max(rect1.y + rect1.height, rect2.y + rect2.height);
      const minX = Math.min(rect1.x, rect2.x);
      const minY = Math.min(rect1.y, rect2.y);
      if (
        maxX - minX <= rect1.width + rect2.width &&
        maxY - minY <= rect1.height + rect2.height
      ) {
        return true;
      } else {
        return false;
      }
    },
    handleDomSelect() {
      const dom_mask = document.querySelector(".mask");
      const rect_select = dom_mask.getClientRects()[0];
      const targetDom = document.querySelector(".tableBox tbody tr");
      document.querySelectorAll(".tableBox tbody tr td").forEach((node) => {
        const rects = node.getClientRects()[0];
        node.classList.remove("active");
        if (this.collide(rects, rect_select) === true) {
          //获取框选的节点
          console.log(node);
          node.classList.add("active");
        }
      });
    },
    resSetXY() {
      this.start_x = 0;
      this.start_y = 0;
      this.end_x = 0;
      this.end_y = 0;
    },
  },
  created() {},
  mounted() {
    this.$nextTick(() => {
      const dom_box = document.querySelector(".tableBox");
      this.box_screen_left = dom_box.getBoundingClientRect().left;
      this.box_screen_top = dom_box.getBoundingClientRect().top;
    });
  },
};
</script>
<style lang="scss" scoped>
.menuBox {
  position: fixed;
}
.mask {
  position: absolute;
  background: #409eff;
  opacity: 0.4;
  z-index: 10000;
}
.tableBox {
  user-select: none;
  width: 500px;
  position: relative;

  thead {
    td {
      cursor: pointer;
    }
  }
  td {
    padding: 20px;
    border: 1px solid #ddd;
    &.active {
      background-color: greenyellow;
    }
  }
}
</style>