<template>
  <div class="container">
    <div class="row">
      <div class="col-auto component-navbar left-area-rwd">
        <Navbar />
      </div>
      <div class="col main-area-rwd vh100scroll">
        <UserProfile :initialUserProfile="userProfile" />
        <div class="userNavbar d-flex">
          <router-link
            class="user-nav-link d-block"
            :to="{ name: 'user', params: { userId: userProfile.id } }"
            >推文</router-link
          >
          <router-link
            class="user-nav-link d-block"
            :to="{
              name: 'user-with-replies',
              params: { userId: userProfile.id },
            }"
            >推文與回覆</router-link
          >
          <router-link
            :to="{ name: 'user-likes', params: { userId: userProfile.id } }"
            class="user-nav-link d-block"
            >喜歡的內容</router-link
          >
        </div>
        <Tweets :initialTweets="tweets" />
      </div>
      <div class="col-auto right-area-rwd">
        <TopFollowersUser
          class="component-top-followers-user"
          :initialUserProfile="userProfile"
          @after-click-delete-following="afterClickDeleteFollowing"
          @after-click-add-following="afterClickAddFollowing"
        />
      </div>
    </div>
  </div>
</template> 

<script>
import Navbar from "./../components/Navbar";
import TopFollowersUser from "./../components/TopFollowersUser";
import UserProfile from "./../components/UserProfile";
import Tweets from "./../components/Tweets.vue";

import userAPI from "./../apis/user";
import { Toast } from "./../utils/helpers";

import { mapState } from "vuex";

export default {
  components: {
    Navbar,
    TopFollowersUser,
    UserProfile,
    Tweets,
  },
  data() {
    return {
      userProfile: {},
      tweets: [],
    };
  },
  watch: {
    initialTweets(newValue) {
      this.tweets = newValue
    },
    initialUserProfile(newValue) {
      this.userProfile = newValue
    }
  },
  computed: {
    ...mapState(["currentUser", "isAuthenticated"]),
  },
  async created() {
    const { userId: userId } = this.$route.params;
    await this.fetchUserProfile(userId);
    this.fetchUserTweets(userId);
  },
  async beforeRouteUpdate(to, from, next) {
    const userId = to.params.userId;
    await this.fetchUserProfile(userId);
    this.fetchUserTweets(userId);
    next();
  },
  methods: {
    async fetchUserProfile(userId) {
      try {
        const response = await userAPI.getUserProfile({ userId });

        this.userProfile = {
          ...this.userProfile,
          ...response.data,
        };

        if (response.status !== 200) {
          throw new Error(response);
        }
      } catch (error) {
        console.log(error);
        Toast.fire({
          icon: "error",
          title: "無法取得使用者資料，請稍後再試",
        });
      }
    },
    async fetchUserTweets(userId) {
      try {
        const response = await userAPI.getUserTweets({ userId });
        console.log(response);

        this.tweets = response.data;
        this.tweets = this.tweets.map((tweet) => {
          return {
            ...tweet,
            User: { ...this.userProfile },
          };
        });

        if (response.status !== 200) {
          throw new Error(response);
        }
      } catch (error) {
        console.log(error);
        Toast.fire({
          icon: "error",
          title: "無法取得推文，請稍後再試",
        });
      }
    },
    afterClickDeleteFollowing(payload) {

      const { userId } = payload
      // console.log('父層被點追縱的userId', userId)
      // console.log('父層當前頁面this.userProfile.id', this.userProfile.id)
      // console.log('currentUser', this.currentUser.id)

      if (userId === this.userProfile.id) {
        //被追蹤減一
        const newFollowersCount = this.userProfile.FollowersCount - 1
        this.userProfile = {
          ...this.userProfile,
          FollowersCount: newFollowersCount
        }
        return
      } else if (this.userProfile.id === this.currentUser.id) {
        //追蹤減一
        const newFollowingCount = this.userProfile.FollowingsCount - 1
        this.userProfile = {
          ...this.userProfile,
          FollowingsCount: newFollowingCount
        }
        return
      }

    },
    afterClickAddFollowing(payload) {
      const { userId } = payload

      if (userId === this.userProfile.id) {
        //被追蹤加一
        const newFollowersCount = this.userProfile.FollowersCount + 1
        this.userProfile = {
          ...this.userProfile,
          FollowersCount: newFollowersCount
        }
        return
      } else if (this.userProfile.id === this.currentUser.id) {
        //追蹤加一
        const newFollowingCount = this.userProfile.FollowingsCount + 1
        this.userProfile = {
          ...this.userProfile,
          FollowingsCount: newFollowingCount
        }
        return
      }
    }
  },
};
</script>

<style scoped>
.user-nav-link {
  text-align: center;
  padding: 15px 0;
  color: #657786;
  /* padding: 15px 40px; */
  width: 120px;
  font-size: 15px;
  font-weight: 700;
}
.user-nav-link:hover {
  color: #ff6600;
}
.userNavbar {
  border: 1px solid #e6ecf0;
  border-top: 0;
}
.active {
  border-bottom: 2px solid #ff6600;
}
</style>