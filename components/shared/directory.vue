<template>
  <div
    v-if="data"
    @dblclick="openExplorer(data)"
    @contextmenu.prevent.stop="handleClick($event, data)"
    class="directory-wrapper mx-10 mt-3"
    :class="isSameIdClipboard && isCutClipboard ? 'active' : ''"
  >
    <img src="~@/assets/images/base/folder.svg" alt="" />
    <p>{{ data.name }}</p>

    <vue-context
      :elementId="'directoryId'"
      :options="options"
      ref="vueSimpleContextMenuDirectory"
      @option-clicked="optionClicked($event)"
    >
    </vue-context>
  </div>
</template>

<script>
// Mixins
import ContextMenu from "~/mixins/contextMenu.js";

export default {
  props: {
    data: {
      required: true,
      type: Object
    }
  },
  mixins: [ContextMenu],
  data() {
    return {
      options: [
        {
          name: "Editar",
          slug: "edit"
        },
        {
          type: "divider"
        },
        {
          name: "Cortar",
          slug: "cut"
        },
        {
          name: "Copiar",
          slug: "copy"
        },
        {
          type: "divider"
        },
        {
          name: "Eliminar",
          slug: "delete"
        },
        {
          type: "divider"
        },
        {
          name: "Permisos",
          slug: "chmod"
        },
        {
          name: "Propietario",
          slug: "chown"
        }
      ]
    };
  },
  computed: {
    isSameIdClipboard() {
      return this.$store.state.idClipboard == this.data.id;
    },
    isCutClipboard() {
      return this.$store.state.typeClipboard == "cut";
    },
    user() {
      return this.$store.state.localStorage.user;
    }
  },
  methods: {
    openExplorer(data) {
      if (this.user.role.name == "admin" && !this.data.readableRoot) {
        this.notEnoughAccess();
        return;
      }
      if (
        this.user.role.name == "user" &&
        !this.data.readableUser &&
        this.data.user.name !== this.$store.state.localStorage.user.name
      ) {
        this.notEnoughAccess();
        return;
      }
      if (this.user.role.name == "guest" && !this.data.readableGuest) {
        this.notEnoughAccess();
        return;
      }

      const {
        readableGuest,
        writableGuest,
        readableRoot,
        writableRoot,
        writableUser,
        readableUser
      } = this.data;

      let rules = {
        readableGuest,
        writableGuest,
        readableRoot,
        writableRoot,
        writableUser,
        readableUser
      };

      this.$store.dispatch("setActivePosition", data.id);
      this.$store.commit("permissions/SET_USER_DATA", this.data.user);
      this.$store.commit("permissions/SET_RULES", rules);

      this.$store.dispatch("openExplorer", data);
    },
    notEnoughAccess() {
      this.$toast.error(
        "No cuentas con los permisos suficientes para entrar a este directorio",
        { duration: 2000 }
      );
    },
    handleClick(event, item) {
      this.$store.dispatch("editor/setType", "folder");
      this.$store.dispatch("editor/setFolder", item);
      this.$refs.vueSimpleContextMenuDirectory.showMenu(event, item);
    },
    optionClicked(event) {
      this.switchOption(event.option.slug);
    }
  }
};
</script>

<style lang="scss" scoped>
.directory-wrapper {
  position: relative;
  display: inline-block;
  cursor: pointer;

  &.active {
    img,
    p {
      opacity: 0.6;
    }
  }
  img {
    width: 40px;
  }
  p {
    position: absolute;
    text-align: center;
    width: 100%;
    font-size: 12px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    // transition: .3s ease-in-out;
    color: $white;
  }
  &:hover {
    p {
      width: 100%;
      overflow: initial;
      text-overflow: initial;
      white-space: initial;
    }
  }
}
</style>
