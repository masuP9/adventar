<template>
  <header class="GlobalHeader">
    <div class="inner">
      <h1 class="logo">
        <nuxt-link to="/"><img src="~/assets/logo.png" alt="Adventar" width="220" height="28"/></nuxt-link>
      </h1>
      <div class="right">
        <no-ssr>
          <span role="button" @click.stop="showDropdown()" class="menuBtn">
            <UserIcon v-if="$store.state.user" class="userIcon" :user="$store.state.user" size="28" />
            <font-awesome-icon v-else icon="bars"></font-awesome-icon>
          </span>
          <div class="dropdown" v-if="isShownDropdown" @click.stop>
            <ul v-if="$store.state.user" class="loginMenu">
              <li class="user">
                <UserIcon class="userIcon" :user="$store.state.user" size="28" />
                {{ $store.state.user.name }}
              </li>
              <li>
                <nuxt-link @click.native="hideDropdown()" to="/new">
                  <font-awesome-icon icon="calendar-plus" />
                  カレンダーを作る
                </nuxt-link>
              </li>
              <li>
                <nuxt-link @click.native="hideDropdown()" :to="`/users/${$store.state.user.id}`">
                  <font-awesome-icon icon="user" /> マイページ
                </nuxt-link>
              </li>
              <li>
                <nuxt-link @click.native="hideDropdown()" to="/setting">
                  <font-awesome-icon icon="cog" /> 設定
                </nuxt-link>
              </li>
              <li>
                <span @click.native="hideDropdown()" role="button" @click="logout()">
                  <font-awesome-icon icon="sign-out-alt" /> ログアウト
                </span>
              </li>
            </ul>
            <ul v-if="!$store.state.user" class="loginMenu">
              <li>
                <span @click.native="hideDropdown()" role="button" @click="login('google')">
                  <font-awesome-icon :icon="['fab', 'google']" /> Google でログイン
                </span>
              </li>
              <li>
                <span @click.native="hideDropdown()" role="button" @click="login('github')">
                  <font-awesome-icon :icon="['fab', 'github']" /> GitHub でログイン
                </span>
              </li>
              <li>
                <span @click.native="hideDropdown()" role="button" @click="login('twitter')">
                  <font-awesome-icon :icon="['fab', 'twitter']" /> Twitter でログイン
                </span>
              </li>
              <li>
                <span @click.native="hideDropdown()" role="button" @click="login('facebook')">
                  <font-awesome-icon :icon="['fab', 'facebook']" /> Facebook でログイン
                </span>
              </li>
            </ul>
            <ul class="generalMenu">
              <li>
                <nuxt-link @click.native="hideDropdown()" to="/archive">
                  <font-awesome-icon icon="calendar-minus" /> 過去のカレンダー
                </nuxt-link>
              </li>
              <li>
                <nuxt-link @click.native="hideDropdown()" to="/help">
                  <font-awesome-icon icon="question-circle" /> ヘルプ
                </nuxt-link>
              </li>
            </ul>
          </div>
        </no-ssr>
      </div>
    </div>
  </header>
</template>

<script lang="ts">
import { Component, Vue } from "nuxt-property-decorator";
import { loginWithFirebase, logoutWithFirebase } from "~/lib/Auth";
import UserIcon from "~/components/UserIcon.vue";

@Component({
  components: { UserIcon }
})
export default class extends Vue {
  isShownDropdown = false;

  mounted() {
    document.addEventListener("click", this.handleClickDocument);
  }

  destroyed() {
    document.removeEventListener("click", this.handleClickDocument);
  }

  handleClickDocument() {
    this.hideDropdown();
  }

  showDropdown() {
    this.isShownDropdown = true;
  }

  hideDropdown() {
    this.isShownDropdown = false;
  }

  login(provider) {
    loginWithFirebase(provider);
  }

  logout() {
    this.$router.push("/");
    logoutWithFirebase();
  }
}
</script>

<style lang="scss" scoped>
.GlobalHeader {
  background-color: #fff;
}
.inner {
  max-width: $content-max-width;
  margin: 0 auto;
  position: relative;
  padding: 5px 12px;
}

.right {
  position: absolute;
  right: 15px;
  top: 18px;
}

.logo {
  margin: 0;
  padding: 10px 0;
  font-size: 24px;
  font-weight: bold;
}
.logo a {
  color: #e4523d;
  text-transform: uppercase;
  text-decoration: none;
}

.logo img {
  width: 165px;
  height: 21px;
}

.menuBtn {
  color: #333;
  cursor: pointer;
  display: block;
  padding-bottom: 10px;
  font-size: 20px;
}

.menuBtn.is-signin {
  font-size: 20px;
}

.menuBtn:hover {
  color: #000;
}

.menuBtn .userIcon {
  margin-right: 5px;
}

.dropdown {
  position: absolute;
  width: 100%;
  z-index: 1;
}

.dropdown ul {
  border: 1px solid #dadada;
  border-radius: 3px;
  background: #fff;
  width: 200px;
  margin: 0;
  padding: 0;
  font-size: 14px;
  float: right;

  &.loginMenu {
    border-radius: 3px 3px 0 0;
  }

  &.generalMenu {
    border-radius: 0 0 3px 3px;
    border-top: none;
  }
}

.dropdown li {
  margin: 0;
  padding: 0;
  list-style: none;
}

.dropdown li.user {
  padding: 5px 10px;
  margin-bottom: 5px;
  background-color: #eaeaea;
}

.dropdown li svg {
  margin-right: 5px;
}

.dropdown li [role="button"],
.dropdown li a {
  display: block;
  color: #666;
  font-size: 13px;
  padding: 10px 10px;
  text-decoration: none;
  cursor: pointer;

  &:hover {
    color: #fff;
    background: #e45541;
  }
}

@media (min-width: $mq-break-small) {
  .inner {
    padding: 20px 12px;
  }

  .right {
    top: 32px;
    right: 12px;
  }

  .logo img {
    width: 220px;
    height: 28px;
  }
}
</style>
