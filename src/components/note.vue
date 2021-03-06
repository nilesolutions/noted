<template>
    <v-card
            class="note mx-auto scroll-y"
            outlined
            shaped
            dark
            color="indigo lighten-2"
            height="100%"
            width="100%"
            @wheel.stop=""
    >
        <v-toolbar absolute dense collapse color="indigo darken">
            <v-menu
                    offset-y
                    top
                    origin="center center"
                    transition="scale-transition">
                <template v-slot:activator="{ on }">
                    <v-app-bar-nav-icon v-on="on"></v-app-bar-nav-icon>
                </template>
                <v-list
                    dark
                    color="indigo darken">
                    <v-list-item link @click="deleteNote" v-touch:tap="deleteNote">
                        <v-list-item-icon class="material-icons">delete</v-list-item-icon>
                        <v-list-item-content>
                            Delete Note
                        </v-list-item-content>
                    </v-list-item>
                    <v-list-item link @click="openSettings" v-touch:tap="openSettings">
                        <v-list-item-icon class="material-icons">settings</v-list-item-icon>
                        <v-list-item-content>
                            Settings
                        </v-list-item-content>
                    </v-list-item>
                    <v-list-item v-if="note.container" link @click="escape">
                        <v-list-item-icon class="material-icons">eject</v-list-item-icon>
                        <v-list-item-content>
                            Escape Container
                        </v-list-item-content>
                    </v-list-item>
                </v-list>
            </v-menu>
            <v-btn icon class="material-icons" v-model="note.editing" @click="toggleEditing">
                edit
            </v-btn>
        </v-toolbar>
        <v-card-text class="static" v-if="!note.editing" v-html="note.text">
        </v-card-text>
        <v-card-text v-if="note.editing" @keydown.stop="">
            <ckeditor v-model="note.text" :editor="editor" @ready="ready" :config="editorConfig"></ckeditor>
        </v-card-text>
        <v-dialog v-model="dialog" width="300">
            <v-card>
                <v-card-title class="headline">
                    Edit Note
                </v-card-title>
                <v-card-text>
                    <v-text-field
                            v-model="note.name"
                            label="Note Name"
                            required
                    ></v-text-field>

                    <v-slider
                            label="Note Order"
                            max="100" v-model.number="note.z"
                            min="0" thumb-label="always"
                    ></v-slider>
                    <v-checkbox label="Lock" v-model="note.locked"></v-checkbox>
                    <v-checkbox label="Grid" v-model="note.grid"></v-checkbox>
                    <v-btn @click="deleteNote" dark class="dangerous" color="red">
                        <v-icon>delete</v-icon>
                    </v-btn>

                </v-card-text>
            </v-card>
        </v-dialog>
    </v-card>
</template>

<script>
    import balloonEditor from '@ckeditor/ckeditor5-build-balloon';

    export default {
        name: 'note',
        props: {
            note: Object,
            anchor: Object,
            container: Object,
        },
        components: {},
        data() {
            if (this.note.z === undefined) {
                this.$set(this.note, 'z', 0)
            }

            return {
                startWithFocus: this.note.editing,
                editing: false,
                clipboardMessage: "",
                dialog: false,
                editor: balloonEditor,
                editorConfig: {
                    link: {
                        decorators: {
                            isExternal: {
                                mode: 'manual',
                                label: 'Open in a new tab',
                                attributes: {
                                    target: '_blank'
                                }
                            }
                        }
                    }
                },
            }
        },
        methods: {
            openSettings() {
                this.dialog = true
            },
            ready(e) {
                e.ui.element.focus()
            },
            escape() {
                this.note.container = false
                this.note.x = this.container.x
                this.note.y = this.container.y
                this.note.width = 200
                this.note.height = 200
                this.anchor.notes.push(this.note)
                this.container.notes = this.container.notes.filter(v => {
                    return v.id !== this.note.id
                })
            },
            toggleEditing() {
                this.$set(this.note, 'editing', !this.note.editing)
            },
            handleClick(item) {
                let link = '<a href="#/notes/' + item.option.note.id + '">' + item.option.name + '</a>'
                this.$set(this.note, 'text', link + '<br>' + this.note.text)
            },
            touch(e) {
                this.dialog = true
                e.stopPropagation()
            },
            deleteNote() {
                if (this.container) {
                    this.escape()
                }
                this.anchor.notes = this.anchor.notes.filter(val => {
                    return val.id !== this.note.id
                })
            },
        },
    }
</script>


<style>
    .note {
        touch-action: none;
        padding-bottom: 16px;
    }

    .note .v-card__text {
        height: 100%;
        padding-top: 30px;
        overflow-y: auto;
    }

    .note .v-card__text.static {
        padding: 50px 25px 25px 25px;
    }
</style>
