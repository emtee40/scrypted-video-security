<template>
  <v-toolbar
    dense
    dark
    :color="color || ($vuetify.theme.dark ? undefined : colors.indigo.darken2)"
    ><v-icon small v-if="icon">{{ icon }}</v-icon
    >&nbsp;&nbsp;
    <slot></slot>
  </v-toolbar>
</template>
<script>
import colors from "vuetify/es5/util/colors";

export default {
  props: ["color", "icon"],

  data() {
    return {
      colors,
    };
  },
};
</script>
