<template>
  <div v-if="user">
    <PageHeader>
      <UserIcon :user="user" size="30" />
      {{ user.name }}
    </PageHeader>

    <main>
      <div class="mainInner">
        <ul class="years">
          <li v-for="y in years" :key="y">
            <nuxt-link
              :class="year === y ? 'is-current' : ''"
              :to="`/users/${user.id}?year=${y}`"
              @click.native="handleClickYear(y)"
              >{{ y }}</nuxt-link
            >
          </li>
        </ul>

        <div v-if="loaded">
          <SectionHeader>作成したカレンダー</SectionHeader>
          <CalendarList class="calendarList" :calendars="myCalendars" />

          <SectionHeader>登録したカレンダー</SectionHeader>
          <ul class="entryList">
            <li v-for="e in myEntries" :key="e.id">
              <font-awesome-icon :icon="['far', 'calendar']" />
              {{ formatEntryDate(year, e.day) }}
              <nuxt-link :to="`/calendars/${e.calendar.id}`">{{ e.calendar.title }}</nuxt-link>
            </li>
          </ul>

          <template v-if="mypage">
            <SectionHeader>iCal</SectionHeader>
            <p>投稿の予定をGoogle Calendarなどに読み込むことができます。</p>
            <input class="icalInput" type="text" @click="handleClickIcalInput" :value="icalUrl" />
          </template>
        </div>
        <div v-else class="loading">
          <font-awesome-icon icon="circle-notch" spin />
        </div>
      </div>
    </main>
  </div>
</template>

<script lang="ts">
import dayjs from "dayjs";
import { Component, Watch, Vue } from "nuxt-property-decorator";
import { User, Calendar, Entry } from "~/types/adventar";
import { getCurrentYear } from "~/lib/Configuration";
import { getUser, listCalendars, listEntries } from "~/lib/GrpcClient";
import PageHeader from "~/components/PageHeader.vue";
import SectionHeader from "~/components/SectionHeader.vue";
import CalendarList from "~/components/CalendarList.vue";
import UserIcon from "~/components/UserIcon.vue";

const dayOfWeek = ["日", "月", "火", "水", "木", "金", "土"];

@Component({
  components: { PageHeader, SectionHeader, CalendarList, UserIcon }
})
export default class extends Vue {
  year: number = getCurrentYear();
  user: User | null = null;
  myCalendars: Calendar[] = [];
  myEntries: Entry[] = [];
  loaded: boolean = false;

  @Watch("year")
  async onChangeYear() {
    if (!this.user) return;
    this.loaded = false;
    this.myCalendars = await listCalendars({ year: this.year, userId: this.user.id });
    this.myEntries = await listEntries({ year: this.year, userId: this.user.id });
    this.loaded = true;
  }

  async mounted() {
    this.year = this.$route.query.year ? Number(this.$route.query.year) : getCurrentYear();
    this.user = await getUser(Number(this.$route.params.id));
    await this.onChangeYear();
  }

  get icalUrl(): string {
    return `${location.origin}/users/${this.user!.id}.ics`;
  }

  get years(): number[] {
    const currentYear = getCurrentYear();
    const years: number[] = [];
    for (let y = 2012; y <= currentYear; y++) {
      years.push(y);
    }
    return years.reverse();
  }

  get mypage(): boolean {
    return this.user && this.$store.state.user && this.user.id === this.$store.state.user.id;
  }

  handleClickIcalInput(event): void {
    event.target.select();
  }

  handleClickYear(year) {
    this.year = year;
  }

  formatEntryDate(year: number, day: number): string {
    const d = dayjs(new Date(year, 11, day));
    return `${d.format("YYYY-MM-DD")}（${dayOfWeek[d.day()]}）`;
  }
}
</script>

<style scoped>
.years {
  list-style: none;
  margin: 0;
  padding: 0;
}

.years li {
  display: inline-block;
  margin: 0 10px 0 0;
}

.years li a {
  display: inline-block;
  font-size: 14px;
  text-decoration: none;
  padding: 5px 10px;
  border-radius: 3px;
}

.years li a.is-current,
.years li a:hover {
  background: #e45541;
  color: #fff;
}

.years li a.is-current {
  pointer-events: none;
}

.calendarList {
  margin-bottom: 40px;
}

.entryList {
  list-style: none;
  margin: 0 0 40px 0;
  padding: 0;
}

.entryList li {
  margin: 5px 0;
  padding: 0;
}

.icalInput {
  padding: 5px;
  font-size: 16px;
  width: 100%;
}

.loading {
  text-align: center;
  font-size: 42px;
  padding: 30px 0;
  color: #e35541;
}
</style>
