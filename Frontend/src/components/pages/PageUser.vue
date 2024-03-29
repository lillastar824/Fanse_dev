<template>
  <b-row class="profile" v-if="user != null">
    <b-col>
      <b-row
        :class="
          'page-header sticky-top' +
          (isScrolled ? ' scrolled bg-white border-bottom ' : '')
        "
      >
        <div class="d-flex align-items-center w-100 px-3">
          <b-link to="/">
            <i class="bi-arrow-left" />
          </b-link>
          <h6 class="p-0 my-3 mx-2 flex-grow-1 overflow-hidden">
            <ui-username :user="user" :asLink="false" :bold="false" />
            <div  no-caret ><span style="font-size: 0.8rem !important;"> 7 Posts | 1.3k Likes </span></div>
          </h6>
          <b-dropdown no-caret right variant="link" v-if="this.$store.state.currentUser">
            <template slot="button-content"
              ><i class="bi-three-dots-vertical text-white"
            /></template>
            <b-dropdown-item @click.prevent="copyLink">{{
              $t("general.copy-link-to-profile")
            }}</b-dropdown-item>
            <b-dropdown-item @click.prevent="addToList(user)">{{
              $t("general.add-to-list")
            }}</b-dropdown-item>
          </b-dropdown>
        </div>
      </b-row>
      <b-row>
        <div class="bg-light cover w-100 position-relative" style="margin-top: -70px;">
          <b-img :src="user.cover + '?rnd=' + rnd" v-if="user.cover != null" />
        </div>
        <div class="subprofile w-100 d-flex">
          <div class="avatar position-relative ml-3 rounded-circle">
            <b-avatar
              :src="user.avatar + '?rnd=' + rnd"
              :text="user.initials"
              :to="user.url"
              size="100px"
            />
            <div style="position:absolute; bottom: 5px; right: 5px">
              <i style="color: #65D164" class="ml-1 bi bi-circle-fill" v-if="user.online_status"></i>
            </div>
          </div>
          <div class="d-flex mx-2 mt-2 ml-auto" style="font-size: 150%">
            <b-link :to="this.$store.state.token ? '/messages/' + user.id : '/signup'" class="mx-2" v-if="!isOwner">
              <i class="bi-chat" />
            </b-link>
            <b-link class="mx-2" @click.prevent="addToList(user)" v-if="!isOwner">
              <i class="bi-star" />
            </b-link>
            <b-link class="mx-2" @click.prevent="copyLink">
              <i class="bi-box-arrow-up-right" />
            </b-link>
            <b-link :to="'/settings'" class="mx-2" v-if="isOwner">
              <i class="bi-gear" />
            </b-link>
            <b-link @click.prevent="tip" class="mx-2" v-if="!isOwner">
              <i class="bi-coin" />
            </b-link>
          </div>
        </div>
      </b-row>
      <b-row class="px-3 mt-3 d-block border-bottom pb-3 overflow-hidden">
          <div class="d-flex flex-float">
            <ui-username :user="user" :asLink="false" />
          </div>
        <div class="text-muted small small-username">
          {{ "@" + user.username }}
          <span v-if="!user.online_status">&nbsp;&middot;&nbsp;&nbsp;Last seen {{user.time_difference}} {{user.hms}} ago</span>
          <span v-else>&nbsp;&middot;&nbsp;&nbsp;Available now</span>
        </div>
        <UiVueShowMoreText
            :text="user.bio"
            :lines="3"
            additional-content-css="font-size:16px;"
            additional-content-expanded-css="font-size:16px;"
            additional-anchor-css="font-size: 12px;"
            @click="change"
          />
        <div v-if="user.location" class="text-muted small">
          {{ user.location }}
        </div>
        <div v-if="user.website" class="small">
          <b-link target="_blank" :href="user.website">{{
            user.website
          }}</b-link>
        </div>
      </b-row>
      <b-row
        class="px-3 mt-3 d-block border-bottom pb-3"
        v-if="!user.isSubscribed && !isOwner"
      >
        <h5 class="mb-3">{{ $t("general.subscription") }}</h5>
        <b-button variant="primary" block @click.prevent="subscribe">
          {{
            $t("general.subscribe-for-x", [
              user.isFree ? $t("general.free") : user.priceFormatted,
            ])
          }}
        </b-button>
      </b-row>
      <b-row
        class="px-3 mt-3 d-block border-bottom pb-3"
        v-if="
          !isOwner &&
          !user.isSubscribed &&
          !user.isFree &&
          user.bundles.length > 0
        "
      >
        <h5 class="mb-3">{{ $t("general.subscription-bundles") }}</h5>
        <b-button
          variant="primary"
          block
          @click.prevent="subscribe(item)"
          v-for="(item, index) in user.bundles"
          :key="index"
        >
          {{ $t("general.get-x-months-for-y-z-off", item.title(user)) }}
        </b-button>
      </b-row>
      <b-row class="d-block" v-if="isOwner">
        <b-nav tabs justified>
          <b-nav-item
            :active="postsType == types.active"
            @click.prevent="updatePosts(types.active)"
            >{{ $t("general.posts") }}</b-nav-item
          >
          <b-nav-item
            :active="postsType == types.scheduled"
            @click.prevent="updatePosts(types.scheduled)"
            >{{ $t("general.scheduled") }}</b-nav-item
          >
          <b-nav-item
            :active="postsType == types.expired"
            @click.prevent="updatePosts(types.expired)"
            >{{ $t("general.expired") }}</b-nav-item
          >
        </b-nav>
      </b-row>
      <ui-posts v-model="posts" />
    </b-col>
  </b-row>
</template>
<style lang="scss" scoped>
.profile {
  .page-header {
    top: -56px;
    transition: top 0.2s;
    color: white;
    .btn,
    a:not(.dropdown-item) {
      padding: 0;
      color: white;
    }
    &.scrolled {
      top: 0;
      margin-bottom: 0;
      color: black;
      .btn,
      a {
        color: black;
      }
    }
  }
  .cover {
    height: 180px;
    &:after {
      position: absolute;
      left: 0;
      right: 0;
      top: 0;
      bottom: 0;
      background: #000;
      opacity: 0.2;
      pointer-events: none;
      content: "";
      z-index: 3;
    }
    img {
      -o-object-fit: cover;
      object-fit: cover;
      position: absolute;
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
    }
  }
  .avatar {
    margin-top: -50px;
    z-index: 3;
  }
}
</style>
<style scoped lang="scss">
@import "~@/assets/scss/_variables.scss";
.nav-tabs .nav-link {
  border: none;
  border-bottom: 2px solid transparent;
  border-radius: 0;
  color: $secondary;
}
.nav-tabs .nav-link.active,
.nav-tabs .nav-item.show .nav-link {
  color: $primary;
  border-color: $primary;
}
</style>
<script>
import Post from "../models/Post";
import User from "../models/User";
import UiPosts from "../ui/UiPosts.vue";
import Payment from "../models/Payment";
import UiUsername from "../ui/UiUsername.vue";
import UiVueShowMoreText from "../ui/UiVueShowMoreText.vue";

export default {
  components: { UiPosts, UiUsername,UiVueShowMoreText },
  data: function () {
    return {
      user: null,
      posts: [],
      scrollPosition: null,
      page: 1,
      hasMore: false,
      isLoading: false,
      rnd: Math.random(),
      postsType: Post.TYPE_ACTIVE,
    };
  },
  computed: {
    isScrolled: function () {
      return this.scrollPosition >= 180;
    },
    isOwner: function () {
      if (this.$store.state.currentUser) return this.user.id == this.$store.state.currentUser.id;
      else return false;
    },
    username: function () {
      return this.$route.params.username;
    },
    types: function () {
      return {
        active: Post.TYPE_ACTIVE,
        scheduled: Post.TYPE_SCHEDULED,
        expired: Post.TYPE_EXPIRED,
      };
    },
  },
  watch: {
    
    username: function (oldV, newV) {
      if (oldV && newV && oldV != newV) {
        this.reset();
        this.loadUser();
      }
    },
  },
  mounted() {
    this.loadUser();
    window.addEventListener("scroll", this.updateScroll);
  },
  methods: {
    change(showAll) {
      console.log(showAll)
    },
    updatePosts(type) {
      this.postsType = type;
      this.reset();
      this.loadPosts();
    },
    reset() {
      this.isLoading = false;
      this.hasMore = false;
      this.page = 1;
      this.posts = [];
    },
    updateScroll() {
      this.scrollPosition = window.scrollY;
      const scrollPosition = window.innerHeight + window.scrollY;
      if (
        document.body.offsetHeight &&
        scrollPosition &&
        document.body.offsetHeight - scrollPosition <= 1000 &&
        !this.isLoading &&
        this.hasMore
      ) {
        this.loadMore();
      }
    },
    loadUser() {
      if (this.$store.state.token){
        this.$get(
          "/users/" + this.username,
          (data) => {
            this.user = new User(data);
            this.loadPosts();
          },
          (errors) => {
            console.log(errors);
          }
        );
      } else {
        this.$get(
          "/users/guest/" + this.username,
          (data) => {
            this.user = new User(data);
            this.loadPosts();
          },
          (errors) => {
            console.log(errors);
          }
        );
      }
      
    },
    loadPosts() {
      this.isLoading = true;
      if (this.$store.state.token){
        this.$get(
          "/posts/user/" +
            this.user.id +
            "?page=" +
            this.page +
            "&type=" +
            this.postsType,
          (data) => {
            let posts = [...this.posts];
            for (let obj of data.data) {
              posts.push(new Post(obj));
            }
            this.posts = posts;
            this.hasMore = data.next_page_url != null;
            this.isLoading = true;
          },
          (errors) => {
            console.log(errors);
          }
        );
      } else{
        this.$get(
          "/posts/guest/" +
            this.user.id +
            "?page=" +
            this.page +
            "&type=" +
            this.postsType,
          (data) => {
            let posts = [...this.posts];
            for (let obj of data.data) {
              posts.push(new Post(obj));
            }
            this.posts = posts;
            this.hasMore = data.next_page_url != null;
            this.isLoading = true;
          },
          (errors) => {
            console.log(errors);
          }
        );
      }
      
    },
    loadMore() {
      if (this.hasMore) {
        this.page = this.page + 1;
        this.loadPosts();
      }
    },
    copyLink() {
      const link = `${process.env.VUE_APP_APP_URL}${this.user.url}`;
      this.$copyText(link);
      this.$bvToast.toast(this.$t("general.link-copied"), {
        autoHideDelay: 2000,
        title: this.$t("general.link"),
        solid: true,
        toaster: "b-toaster-bottom-left",
      });
    },
    addToList(user) {
      if (this.$store.state.token) this.$store.state.addToListUser = user;
      else window.location.href="/signup"
    },
    subscribe(bundle) {
      if (!this.$store.state.currentUser) {
        window.location.href = "/signup"
      } else if (this.user.isFree) {
        this.$post(
          "/subscribe/" + this.user.id,
          {},
          () => {
            this.reset();
            this.loadUser();
          },
          (errors) => {
            console.log(errors);
          }
        );
      } else {
        this.$buyItem({
          type: Payment.TYPE_SUBSCRIPTION_NEW,
          user: this.user,
          bundle: bundle,
        });
      }
    },
    tip() {
      if (this.$store.state.token) {
        this.$buyItem({
        type: Payment.TYPE_TIP,
        user: this.user,
      });
      } else {
        window.location.href="/?next='/adminuser'"
      }
      
    },
  },
};
</script>
