<template>
  <div class="v-upload" :class="{ uploading: Object.keys(files).length > 0, disabled }">
    <input
      :disabled="disabled"
      class="select"
      type="file"
      ref="select"
      :multiple="multiple"
      @change="filesChange($event.target.files)" />

    <div class="dropzone" :class="{ 'smaller': small }">
      <div class="icon">
        <i class="material-icons">cloud_upload</i>
      </div>
      <div class="info">
        <p class="name">{{ $t('drop_files') }}</p>
        <p class="file-info no-wrap">{{ $t("max_size", { size: $helpers.filesize($store.state.serverInfo.maxUploadSize) }) }}</p>
      </div>
      <div class="buttons">
        <i
          v-tooltip="$t('select_from_device')"
          @click="$refs.select.click()" class="material-icons select">devices</i>
      </div>
    </div>
    <transition-group tag="ol" name="list">
      <li
        class="list-item"
        v-for="(file, id) in files"
        :key="id">
        <v-progress-ring
          class="icon"
          :progress="file.progress"
          :icon="file.error !== null ? 'cloud_off' : (file.progress === 100 ? 'cloud_done' : 'cloud_upload')"
          :color="file.error !== null ? 'danger' : (file.progress === 100 ? 'success' : 'accent')"
          :stroke="file.progress === 100 ? 0 : 2" />
        <div class="info">
          <p class="name no-wrap">{{ file.name }}</p>
          <p class="file-info no-wrap">{{ file.size }} <span v-if="file.progress && file.progress !== 100" class="progress">{{ file.progress }}%</span></p>
        </div>
      </li>
    </transition-group>

    <input
      :disabled="disabled"
      class="drop"
      type="file"
      ref="drop"
      :multiple="multiple"
      @click.prevent
      @change="filesChange($event.target.files)" />
  </div>
</template>

<script>
import filesize from "filesize";

export default {
  name: "v-upload",
  props: {
    multiple: {
      type: Boolean,
      default: true
    },
    small: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      files: {}
    };
  },
  methods: {
    filesChange(fileList) {
      if (!fileList || !fileList.length) return;

      Array.from(fileList).forEach(this.save);
    },
    save(file) {
      const id = this.$helpers.shortid.generate();
      const formData = new FormData();
      const { name, size, type } = file;

      formData.append("data", file, name);

      this.files = {
        [id]: {
          name,
          size: filesize(size),
          type,
          progress: 0,
          error: null
        },
        ...this.files
      };

      this.$api
        .uploadFiles(formData, ({ loaded, total }) => {
          const progress = Math.round((loaded * 100) / total);
          this.files[id].progress = progress;
        })
        .then(res => res.data)
        .then(res => {
          this.$emit("upload", {
            ...this.files[id],
            data: res
          });

          // reset the inputs
          this.$refs.select.value = "";
          this.$refs.drop.value = "";
        })
        .catch(error => {
          this.files[id].error = error;
          this.$events.emit("error", {
            notify: this.$t("something_went_wrong_body"),
            error
          });
        });
    }
  }
};
</script>

<style lang="scss" scoped>
.v-upload {
  position: relative;
  background-color: var(--white);
  width: 100%;
  height: var(--width-medium);

  &.disabled {
    background-color: var(--body-background);
    cursor: not-allowed;
  }
}

.dropzone {
  position: relative;

  .buttons {
    position: absolute;
    top: 22px;
    right: 20px;

    > * {
      display: inline-block;
      cursor: pointer;
      transition: color var(--fast) var(--transition);
      user-select: none;

      &:hover {
        transition: none;
      }
    }
  }
}

input.drop {
  opacity: 0;
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

input.select {
  display: none;
}

.v-upload:not(.uploading) .dropzone {
  color: var(--dark-gray);
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  user-select: none;
  transition: var(--fast) var(--transition);
  transition-property: border-color, color;
  border: var(--input-border-width) dashed var(--lighter-gray);
  border-radius: var(--border-radius);

  .icon i {
    font-size: 100px;
    color: var(--lighter-gray);
    margin-bottom: -6px;
  }

  p {
    color: currentColor;

    &:first-of-type {
      font-size: 34px;
      font-weight: 300;
    }
  }

  .info {
    text-align: center;
    color: var(--lighter-gray);
  }

  .file-info {
    text-align: center;
    color: var(--lighter-gray);
    margin-top: 8px;
  }

  .buttons > * {
    color: var(--lighter-gray);

    &:hover {
      color: var(--dark-gray);
    }
  }

  .dragging & {
    transition: border-color var(--slow) var(--transition);
    border-color: var(--darkest-gray);
  }

  &.smaller {
    .icon i {
      font-size: 60px;
    }

    p {
      &:first-of-type {
        font-size: 22px;
      }
    }
  }
}

.v-upload.uploading {
  display: flex;
  flex-direction: column;

  .dragging & ol {
    transition: border-color var(--slow) var(--transition);
    border-color: var(--darkest-gray);
  }

  .dropzone {
    background-color: var(--darkest-gray);
    border-top-left-radius: var(--border-radius);
    border-top-right-radius: var(--border-radius);
    padding: 10px 20px;
    padding-right: 50px;
    border: var(--input-border-width) solid var(--darkest-gray);
    color: var(--white);
    flex-shrink: 0;

    .info {
      overflow: hidden;
    }

    .file-info {
      color: var(--light-gray);
    }

    .icon {
      width: 48px;
      height: 48px;
      display: flex;
      justify-content: center;
      align-items: center;

      i {
        width: 40px;
        height: 40px;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-grow: 0;
        flex-shrink: 0;
        border-radius: 50%;
        border: 2px solid var(--white);
        color: var(--white);
      }
    }

    .buttons {
      & > * {
        color: var(--light-gray);

        &:hover {
          color: var(--white);
        }
      }
    }
  }

  ol {
    flex-grow: 1;
    border: var(--input-border-width) dashed var(--lighter-gray);
    border-top: 0;
    border-bottom-left-radius: var(--border-radius);
    border-bottom-right-radius: var(--border-radius);
    padding: 0 20px;
    list-style: none;
    overflow: scroll;
    -webkit-overflow-scrolling: touch;
  }

  .dropzone,
  li {
    display: flex;
    align-items: center;
    height: 70px;

    .icon {
      margin-right: 5px;
    }
  }

  li {
    padding: 10px 0;
    background-color: var(--white);

    &:not(:last-of-type) {
      border-bottom: 1px solid var(--lightest-gray);
    }

    .file-info {
      color: var(--gray);
    }
  }
}

.v-upload.disabled .buttons {
  pointer-events: none;
}

.dragging input.drop {
  pointer-events: auto;
}

.list-item {
  display: inline-block;

  .info {
    overflow: hidden;
  }
}

.list-enter-active,
.list-leave-active {
  transition: all var(--slow) var(--transition);
}

.list-enter,
.list-leave-to {
  opacity: 0;
}

.list-move {
  transition: all var(--slow) var(--transition);
}
</style>
