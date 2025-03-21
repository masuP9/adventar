<template>
  <div>
    <template v-if="calendar">
      <header class="header" :style="{ backgroundColor: calendarColor }">
        <div class="inner">
          <h2 class="title">{{ title }}</h2>
          <div>登録数 {{ calendar.entries.length }}/25人</div>
          <div>
            作成者
            <nuxt-link class="owner" :to="`/users/${calendar.owner.id}`">
              <UserIcon :user="calendar.owner" size="22" />
              {{ calendar.owner.name }}
            </nuxt-link>
          </div>
          <nuxt-link class="editBtn" :to="`/calendars/${calendar.id}/edit`" v-if="isOwnCalendar(calendar)">
            <font-awesome-icon icon="edit"></font-awesome-icon> 編集
          </nuxt-link>
        </div>
      </header>
      <main>
        <div class="mainInner">
          <div class="description" v-html="descriptionHtml"></div>
          <CalendarTable
            :calendar="calendar"
            :currentUser="$store.state.user"
            :onCreateEntry="handleCreateEntry"
            :onUpdateEntry="handleUpdateEntry"
            :onDeleteEntry="handleDeleteEntry"
          ></CalendarTable>
          <EntryList :calendar="calendar"></EntryList>
        </div>
      </main>
    </template>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "nuxt-property-decorator";
import MarkdownIt from "markdown-it";
import { getCalendar, createEntry, updateEntry, deleteEntry } from "~/lib/GrpcClient";
import * as JsonApiClient from "~/lib/JsonApiClient";
import { calendarColor } from "~/lib/utils/Colors";
import { Calendar } from "~/types/adventar";
import { getToken } from "~/lib/Auth";
import UserIcon from "~/components/UserIcon.vue";
import CalendarTable from "~/components/CalendarTable.vue";
import EntryList from "~/components/EntryList.vue";
import { Entry } from "~/types/adventar";

@Component({
  components: { UserIcon, CalendarTable, EntryList }
})
export default class extends Vue {
  calendar: Calendar | null = null;

  head() {
    if (this.calendar === null) return {};

    return {
      title: `${this.title} - Adventar`,
      meta: [
        { hid: "description", name: "description", content: this.calendar.description },
        { hid: "og:description", property: "og:description", content: this.calendar.description },
        { hid: "og:title", property: "og:title", content: `${this.title} - Adventar` }
      ],
      link: [{ rel: "alternate", type: "application/rss+xml", href: `/calendars/${this.calendar.id}.rss` }]
    };
  }

  async asyncData({ params, error }) {
    if (process.server) {
      let calendar: Calendar;
      try {
        calendar = await JsonApiClient.getCalendar(params.id);
      } catch (err) {
        if (err.response) {
          error({ statusCode: err.response.status });
        } else {
          console.error(err);
          error({ statusCode: 500 });
        }

        return;
      }
      return { calendar };
    }
  }

  async mounted() {
    const clanedarId = Number(this.$route.params.id);
    if (this.calendar && Number(this.calendar.id) !== clanedarId) {
      this.calendar = null;
    }
    // TODO: 404 if not found
    this.calendar = await getCalendar(clanedarId);
  }

  async refetchCalendar() {
    this.calendar = await getCalendar(this.calendar!.id);
  }

  async handleCreateEntry(day: number): Promise<Entry> {
    const token = await getToken();
    const calendarId = this.calendar!.id;
    const entry = await createEntry({ calendarId, day, token });
    await this.refetchCalendar();
    return entry;
  }

  async handleUpdateEntry(entryId: number, { comment, url }: { comment: string; url: string }): Promise<void> {
    const token = await getToken();
    await updateEntry({ entryId, comment, url, token });
    await this.refetchCalendar();
  }

  async handleDeleteEntry(entryId: number): Promise<void> {
    const token = await getToken();
    await deleteEntry({ entryId, token });
    this.calendar = await getCalendar(this.calendar!.id);
    await this.refetchCalendar();
  }

  isOwnCalendar(calendar: Calendar): boolean {
    if (!this.$store.state.user) return false;
    if (!calendar.owner) return false;
    return calendar.owner.id === this.$store.state.user.id;
  }

  get calendarColor(): string {
    return calendarColor(this.calendar!.id);
  }

  get title(): string {
    if (this.calendar === null) {
      return "Adventar";
    }
    return `${this.calendar.title} Advent Calendar ${this.calendar.year}`;
  }

  get descriptionHtml(): string {
    if (!this.calendar || !this.calendar.description) return "";
    return MarkdownIt({ linkify: true, breaks: true }).render(this.calendar.description);
  }
}
</script>

<style lang="scss" scoped>
.mainInner {
  padding: 0 0 30px 0;
}

.header {
  color: #fff;
}

.header > .inner {
  max-width: $content-max-width;
  padding: 30px 12px;
  margin: 0 auto;
  position: relative;
}

.title {
  margin: 0 0 20px 0;
  font-size: 20px;
  font-weight: bold;
}

.owner {
  text-decoration: none;
  color: #fff;
}

.editBtn {
  padding: 10px 20px;
  font-size: 12px;
  display: inline-block;
  border-radius: 5px;
  cursor: pointer;
  text-decoration: none;
  border: none;
  outline: none;
  background: rgba(255, 255, 255, 0.85);
  color: #333;
  position: absolute;
  right: 12px;
  bottom: 30px;
}

.description {
  padding: 5px 10px;
  word-wrap: break-word;
}

@media (min-width: $mq-break-small) {
  .title {
    font-size: 24px;
  }
}
</style>
