<template>
  <div id="app" @scroll="SwitchPage">
    <div id="placeholder" :style="{height: total_height + 'px'}"></div>
    <div id="cover" v-bind:style="{width: cover_size, height: cover_size}"
         v-bind:class="{top_left: (stage % 4 == 0 && stage % 2 == 0),
                        top_right: (stage % 4 == 2 && stage % 2 == 0),
                        bottom_right: (stage % 4 == 1 && stage % 2 == 1),
                        bottom_left: (stage % 4 == 3 && stage % 2 == 1)}">
      <div id="cover-title" v-bind:style="{color: title_color}"
           v-bind:class="{title_top_left: (stage % 4 == 0 && stage % 2 == 0),
                          title_top_right: (stage % 4 == 2 && stage % 2 == 0),
                          title_bottom_right: (stage % 4 == 1 && stage % 2 == 1),
                          title_bottom_left: (stage % 4 == 3 && stage % 2 == 1)}"
        >{{title}}</div>
    </div>
    <div id="main-view">
      <router-view></router-view>
    </div>
    <img src="./assets/keyboard_arrow_down.svg" id="scroll-indicator"
         v-bind:style="{bottom: indicator_pos}"
         v-bind:class="{'attention-indicator': indicator_pos == '6.18vh'}"></img>
  </div>
</template>
<script>
var PagesHandler = {
  Init: function(ChangeRouter, ChangeTitle, ChangeSize, ChangeTotalHeight) {
    var height = document.body.clientHeight;
    var interval_ = height * 0.382;
    var animation_ = height;
    /* ----------
        interval
       ----------

        animation

       ----------
        interval
       ----------
          ...    */
    var pages_ = new Array();
    var stage_ = -1;
    var title_stage = -1;
    var diagonal_ =
        Math.ceil(Math.sqrt(Math.pow(document.body.clientWidth, 2) +
                            Math.pow(animation_, 2)));
    var distance = function(now_scroll) {
      return now_scroll * 4*diagonal_ / animation_;
      // From animation_/2 to 2*diagonal
    }
    var SwitchRouter = function(step) {
      if (stage_ != step) {
        ChangeRouter(pages_[step].router);
        stage_ = step;
      }
    }
    var SwitchTitle = function(step) {
      if (title_stage != step) {
        ChangeTitle(pages_[step].title, pages_[step].color);
        title_stage = step;
      }
    }
    this.Push = function(page) {
      pages_.push(page);
      ChangeTotalHeight(pages_.length * (height + interval_) - interval_);
    }
    this.SwitchPage = function(scroll) {
      var step = Math.floor(scroll / (interval_ + animation_));
      scroll = scroll % (interval_ + animation_);
      // Divide steps
      if (0 <= scroll && scroll < interval_) {
        // Interval
        SwitchRouter(step);
        SwitchTitle(step);
        ChangeSize("0px");
        return step * 2;
      }else if (interval_ <= scroll &&
                scroll < interval_ + animation_/2) {
        // Start animation
        SwitchRouter(step);
        SwitchTitle(step + 1);
        ChangeSize(Math.max(distance(scroll - interval_), 0) + "px");
        return step * 2;
      }else if (interval_ + animation_/2 <= scroll &&
                scroll < interval_ + animation_) {
        // End animation
        SwitchRouter(step + 1);
        SwitchTitle(step + 1);
        ChangeSize(Math.max(distance(interval_ + animation_ - scroll), 0) + "px");
        return step * 2 + 1;
      }
    }
  }
}
export default {
  name: 'app',
  data: function() {
    return {
      cover_size: "0px",
      indicator_pos: "6.18vh",
      stage: 0,
      title: "",
      title_color: "#000",
      total_height: 0
    };
  }, methods: {
    SwitchPage: function(e) {
      this.stage = PagesHandler.SwitchPage(e.srcElement.scrollTop);
      this.indicator_pos = Math.max(6.18 - 10*e.srcElement.scrollTop /
                                    (0.382*document.body.clientHeight), -3.82) +
                           "vh";
    }
  }, mounted: function() {
    PagesHandler.Init((router) => { this.$router.replace(router); },
                      (title, color) => {
                        this.title = title,this.title_color = color; },
                      (size) => { this.cover_size = size; },
                      (height) => { this.total_height = height; });
    PagesHandler.Push({router: "/", title: "", color: "#000"});
    PagesHandler.Push({router: "/introduction", title: "我们是谁", color: "#ffcdd2"});
    PagesHandler.Push({router: "/history", title: "我们做过什么", color: "#c8e6c9"});
    // Three pages now
    PagesHandler.SwitchPage(0);
  }
}
</script>
<style scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin: 0;
  height: 100vh;
  overflow: auto;
}
#placeholder {
  width: 100%;
}
#main-view {
  position: fixed;
  top: 0;
  left: 0;
  z-index: -2;
}
#cover {
  position: fixed;
  z-index: -1;
  border-radius: 100%;
  background-color: #fff;
  box-shadow: 0 1px 6px 0 rgba(32,33,36,0.28);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}
.top_left {
  top: 0;
  left: 0;
  transform: translate(-50%, -50%);
}
.top_right {
  top: 0;
  right: 0;
  transform: translate(50%, -50%);
}
.bottom_left {
  bottom: 0;
  left: 0;
  transform: translate(-50%, 50%);
}
.bottom_right {
  bottom: 0;
  right: 0;
  transform: translate(50%, 50%);
}
#cover-title {
  position: relative;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  font-size: 23.6vh;
}
.title_top_left {
  transform: translate(50%, 50%);
}
.title_top_right {
  transform: translate(-50%, 50%);
}
.title_bottom_left {
  transform: translate(50%, -50%);
}
.title_bottom_right {
  transform: translate(-50%, -50%);
}
#scroll-indicator {
  position: fixed;
  left: 0;
  right: 0;
  margin: auto;
  height: 3.82vh;
}
.attention-indicator {
  animation-name: up-to-down;
  animation-duration: 0.785s;
  animation-direction: alternate;
  animation-iteration-count: infinite
}
@keyframes up-to-down {
  0% {
    bottom: 6.18vh;
  }
  62% {
    bottom: 6.18vh;
  }
  100% {
    bottom: 4.72vh;
  }
}
</style>
