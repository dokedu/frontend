<template>
  <header
    class="flex h-screen w-full max-w-[230px] select-none flex-col justify-between border-r border-stone-100 bg-stone-50 print:hidden"
  >
    <div class="relative flex flex-col">
      <div class="flex items-center justify-between px-3 py-3 pb-1.5">
        <div class="flex flex-1 justify-between rounded-md p-1 hover:bg-strong" @click="visibleAppSwitcher = true">
          <div class="flex items-center gap-3">
            <div class="rounded-lg border border-stone-100 p-1.5">
              <component v-if="app" :is="app.icon" class="stroke-stone-500" :size="12" />
            </div>
            <div class="text-sm text-stone-700 transition-all duration-100 hover:text-stone-950">{{ app?.name }}</div>
          </div>
          <div class="rounded-md border border-stone-200 p-1.5 transition-all hover:bg-stone-200">
            <grip :size="16" class="stroke-stone-700" />
          </div>
        </div>
      </div>
      <div class="flex flex-col gap-1 p-3">
        <router-link
          v-for="link in app?.links"
          :to="{ name: link.route }"
          class="flex items-center gap-3 rounded-md p-1 px-3 text-stone-500 transition-all duration-100 hover:bg-stone-100 hover:text-stone-950"
          active-class=""
          :class="{ '!bg-stone-200 text-stone-900': isLinkActive(link) }"
        >
          <component
            :is="link.icon"
            class="stroke-stone-500"
            :size="18"
            :class="{ '!stroke-stone-900': isLinkActive(link) }"
          />
          <div class="text-sm">{{ link.name }}</div>
        </router-link>
      </div>
      <div ref="appswitcher" v-show="visibleAppSwitcher" class="absolute w-full p-1">
        <div class="flex w-full flex-col gap-1 rounded-lg bg-white p-1 shadow-md">
          <div
            v-for="_app in enabledApps"
            class="flex items-center gap-3 rounded-lg border border-white p-2 text-sm hover:bg-stone-100"
            :class="activeApp === _app.id ? `!border-stone-200 bg-stone-100 hover:!bg-stone-100` : ''"
            @click="switchApp(_app.id)"
          >
            <component
              :is="_app.icon"
              class="stroke-stone-500"
              :size="20"
              :class="activeApp === _app.id ? `!stroke-stone-900` : ''"
            />
            <span class="text-stone-500" :class="activeApp === _app.id ? `!text-stone-900` : ''">{{ _app.name }}</span>
          </div>
        </div>
      </div>
    </div>
    <div class="px-1 py-4">
      <div
        class="flex items-center gap-3 rounded-md p-1 px-3 text-stone-500 transition-all duration-100 hover:bg-stone-100 hover:text-stone-950"
      >
        <globe class="stroke-stone-500" :size="18" />
        <select
          name="language"
          id="lang"
          class="block w-full rounded-lg border border-stone-200 bg-stone-50 px-2 py-1 text-sm"
          v-model="language"
        >
          <option value="en">English</option>
          <option value="de">Deutsch</option>
        </select>
      </div>

      <div
        class="flex items-center gap-3 rounded-md p-1 px-3 text-stone-500 transition-all duration-100 hover:bg-stone-100 hover:text-stone-950"
        @click="signOut"
      >
        <log-out class="stroke-stone-500" :size="18" />
        <div class="text-sm">{{ $t("log_out") }}</div>
      </div>
    </div>
  </header>
</template>
<script setup lang="ts">
import { computed, FunctionalComponent, ref } from "vue";
import {
  CopyCheck,
  Grid,
  Pen,
  PieChart,
  Users,
  Grip,
  Star,
  Clock,
  Folder,
  Wrench,
  Users2,
  Trash2,
  HardDrive,
  LogOut,
  Globe,
} from "lucide-vue-next";
import { Tag } from "lucide-vue-next";
import { onClickOutside, useStorage } from "@vueuse/core";
import { useRoute, useRouter } from "vue-router";
import { UserSquare } from "lucide-vue-next";
import { useMutation } from "@urql/vue";
import { graphql } from "@/gql";
import { watch } from "vue";
import i18n from "@/i18n";
import { useI18n } from "vue-i18n";
import { UserLanguage } from "@/gql/graphql";
import { Landmark } from "lucide-vue-next";
import { Settings } from "lucide-vue-next";
import { School } from "lucide-vue-next";

const { t } = useI18n();

const visibleAppSwitcher = ref<boolean>(false);
const activeApp = useStorage("active_app", "drive");

const route = useRoute();
const router = useRouter();

interface AppLink {
  icon: FunctionalComponent;
  name: string;
  route: string;
}

interface App {
  id: string;
  icon: FunctionalComponent;
  name: string;
  links: AppLink[];
}

const appswitcher = ref();

onClickOutside(appswitcher, () => {
  visibleAppSwitcher.value = false;
});

function isLinkActive(link: AppLink) {
  const name = route.name || "";
  // @ts-expect-error
  return name.startsWith(link.route);
}

const enabledAppList = useStorage("enabled_apps", []);

const app = computed(() => apps.value.find((el) => el.id === activeApp.value) || null);

const enabledApps = computed(() => {
  // @ts-expect-error
  return apps.value.filter((el: { id: string }) => enabledAppList.value.includes(el.id));
});

const apps = computed<App[]>(() => [
  {
    id: "record",
    icon: Pen,
    name: "Dokumentation",
    links: [
      {
        // icon: "file-check-02",
        icon: Pen,
        name: t("entry", 2),
        route: "record-entries",
      },
      {
        // icon: "users-01",
        icon: Users,
        name: t("student", 2),
        route: "record-students",
      },
      // {
      //   // icon: "flag-04",
      //   icon: Flag,
      //   name: "Goals",
      //   route: "home",
      // },
      {
        // icon: "grid-01",
        icon: Grid,
        name: t("project", 2),
        route: "record-projects",
      },
      {
        // icon: "check-done-01",
        icon: CopyCheck,
        name: t("competence", 2),
        route: "record-competences",
      },
      {
        icon: PieChart,
        name: t("report", 2),
        route: "record-reports",
      },
      {
        icon: Tag,
        name: t("tag", 2),
        route: "record-tags",
      },
    ],
  },
  {
    id: "drive",
    icon: Folder,
    name: "Drive",
    links: [
      {
        icon: HardDrive,
        name: "My Drive",
        route: "drive-my-drive",
      },
      {
        icon: HardDrive,
        name: "Shared Drives",
        route: "admin-users",
      },
      {
        icon: Users2,
        name: "Shared with me",
        route: "admin-users",
      },
      {
        icon: Clock,
        name: "Recent",
        route: "admin-users",
      },
      {
        icon: Star,
        name: "Starred",
        route: "admin-users",
      },
      {
        icon: Trash2,
        name: "Trash",
        route: "admin-users",
      },
    ],
  },
  {
    id: "admin",
    icon: Wrench,
    name: "Admin",
    links: [
      {
        // icon: "👥",
        icon: Settings,
        name: t("general"),
        route: "admin-general",
      },
      {
        // icon: "👥",
        icon: Users,
        name: t("user", 2),
        route: "admin-users",
      },
      {
        // icon: "👥",
        icon: Landmark,
        name: t("billing"),
        route: "admin-billing",
      },
    ],
  },
  {
    id: "my_school",
    icon: School,
    name: "My school",
    links: [
      {
        // icon: "👥",
        icon: UserSquare,
        name: t("student", 2),
        route: "admin-students",
      },
      {
        icon: CopyCheck,
        name: t("competence", 2),
        route: "record-competences",
      },
    ],
  },
]);

function switchApp(appId: string | null = null) {
  if (appId) {
    activeApp.value = appId;
    visibleAppSwitcher.value = false;

    // Reroute to first link of app
    const app = apps.value.find((el) => el.id === appId);
    if (app) {
      router.push({ name: app.links[0].route });
    }
    return;
  }
  // start at first app of index is out of bounds
  const nextIndex = (enabledApps.value.findIndex((el) => el.id === activeApp.value) + 1) % enabledApps.value.length;
  activeApp.value = enabledApps.value[nextIndex].id;
}

const { executeMutation: signOutMutation } = useMutation(
  graphql(`
    mutation signOut {
      signOut
    }
  `)
);

async function signOut() {
  await signOutMutation({});

  // Clear the token and enabled_apps from local storage
  localStorage.removeItem("token");
  localStorage.removeItem("enabled_apps");

  // TODO: hack to ensure urql cache is cleared
  location.reload();
}

const { executeMutation: updateLanguage } = useMutation(
  graphql(`
    mutation updateUserLanguage($language: UserLanguage!) {
      updateUserLanguage(language: $language) {
        id
        language
      }
    }
  `)
);

// Using the language from the local storage
// If undefined we set it to english
const language = useStorage("language", "en");

const languageOptions: { [key: string]: UserLanguage } = {
  de: UserLanguage.De,
  en: UserLanguage.En,
};

watch(language, async () => {
  i18n.global.locale.value = language.value as "en" | "de";

  // Update the language in the backend
  await updateLanguage({ language: languageOptions[language.value] });
});
</script>
