<template>
  <q-page class="relative-position">
    <q-scroll-area class="absolute full-width full-height">
      <div class="q-py-lg q-px-md row items-end q-col-gutter-md">
        <div class="col">
          <q-input
            bottom-slots
            class="new-qweet"
            v-model="newQweetContent"
            label="What's happening?"
            counter
            maxlength="280"
            dense
            autogrow
          >
            <template v-slot:before>
              <q-avatar size="xl">
                <img src="profile_square.png" />
              </q-avatar>
            </template>
          </q-input>
        </div>
        <div class="col col-shrink">
          <q-btn
            @click="addNewQweet"
            :disable="!newQweetContent"
            class="q-mb-lg"
            unelevated
            no-caps
            rounded
            color="primary"
            label="Qweet"
          />
        </div>
      </div>

      <q-separator size="10px" color="grey-2" class="divider" />

      <q-list separator>
        <transition-group
          appear
          enter-active-class="animated fadeIn slow"
          leave-active-class="animated fadeOut slow"
        >
          <q-item v-for="qweet in qweets" :key="qweet.id" class="qweet q-py-md">
            <q-item-section avatar top>
              <q-avatar size="xl">
                <img src="profile_square.png" />
              </q-avatar>
            </q-item-section>

            <q-item-section>
              <q-item-label class="text-subtitle1"
                ><strong>Peter Barrow</strong
                ><span class="text-grey-7">
                  @peter_handle <br class="lt-md" />
                  &bull;{{ qweet.date | relativeDate }}
                </span></q-item-label
              >
              <q-item-label class="qweet-content text-body1">
                {{ qweet.content }}
              </q-item-label>
              <div class="qweet-icons row justify-between q-mt-sm">
                <q-btn
                  colors="grey"
                  icon="far fa-comment"
                  size="sm"
                  flat
                  round
                />
                <q-btn
                  colors="grey"
                  icon="fas fa-retweet"
                  size="sm"
                  flat
                  round
                />
                <q-btn
                  @click="toggleLiked(qweet)"
                  colors="grey"
                  :color="qweet.liked ? 'pink' : 'grey'"
                  :icon="qweet.liked ? 'fas fa-heart' : 'far fa-heart'"
                  size="sm"
                  flat
                  round
                />
                <q-btn
                  @click="deleteQweet(qweet)"
                  colors="grey"
                  icon="fas fa-trash"
                  size="sm"
                  flat
                  round
                />
              </div>
            </q-item-section>
          </q-item>
        </transition-group>
        <q-separator inset="item" />
      </q-list>
    </q-scroll-area>
  </q-page>
</template>

<script>
import db from "src/boot/firebase";
import { formatDistance } from "date-fns";
export default {
  name: "PageHome",
  data() {
    return {
      newQweetContent: "",
      qweets: [
        // {
        //   id: 1,
        //   content: "Do it for the children",
        //   date: 1616098727910,
        //   liked: true
        // },
        // {
        //   id: 2,
        //   content:
        //     "Lorem ipsum dolor sit amet consectetur adipisicing elit. Dolor, amet ratione? Cum nihil voluptatum eius iure illo repudiandae recusandae, quas quasi doloremque, nam quibusdam quidem id magnam aliquid dolore soluta",
        //   date: 1616098792249,
        //   liked: false
        // }
      ]
    };
  },
  filters: {
    relativeDate(value) {
      return formatDistance(value, new Date());
    }
  },
  methods: {
    addNewQweet() {
      let newQweet = {
        content: this.newQweetContent,
        date: Date.now(),
        liked: false
      };
      // this.qweets.unshift(newQweet);
      // Add a new document with a generated id.
      db.collection("qweets")
        .add(newQweet)
        .then(docRef => {
          console.log("Document written with ID: ", docRef.id);
        })
        .catch(error => {
          console.error("Error adding document: ", error);
        });
      this.newQweetContent = "";
    },
    deleteQweet(qweet) {
      db.collection("qweets")
        .doc(qweet.id)
        .delete()
        .then(() => {
          console.log("Document successfully deleted!");
        })
        .catch(error => {
          console.error("Error removing document: ", error);
        });
    },
    toggleLiked(qweet) {
      let updtQweet = db.collection("qweets").doc(qweet.id);

      // Set the "capital" field of the city 'DC'
      updtQweet
        .update({
          liked: !qweet.liked
        })
        .then(() => {
          console.log("Document successfully updated!");
        })
        .catch(error => {
          // The document probably doesn't exist.
          console.error("Error updating document: ", error);
        });
      // console.log("toggleLiked", qweet);
    }
  },
  mounted() {
    db.collection("qweets")
      .orderBy("date")
      .onSnapshot(snapshot => {
        snapshot.docChanges().forEach(change => {
          let qweetChange = change.doc.data();
          qweetChange.id = change.doc.id;
          if (change.type === "added") {
            console.log("New qweet: ", qweetChange);
            this.qweets.unshift(qweetChange);
          }
          if (change.type === "modified") {
            console.log("Modified qweet: ", qweetChange);
            let index = this.qweets.findIndex(
              qweet => qweet.id === qweetChange.id
            );
            Object.assign(this.qweets[index], qweetChange);
          }
          if (change.type === "removed") {
            console.log("Removed qweet: ", qweetChange);
            let index = this.qweets.findIndex(
              qweet => qweet.id === qweetChange.id
            );
            this.qweets.splice(index, 1);
          }
        });
      });
  }
};
</script>

<style lang="sass">
.new-qweet
  textarea
    font-size:19px
    line-height: 1.4 !important

.divider
  border-top: 1px solid
  border-bottom: 1px solid
  border-color: $grey-4

.qweet-content
  white-space: pre-line

.qweet-icons
  margin-left: -5px

.qweet:not(:first-child)
  border-top: 1px solid rgba(0,0,0, 0.12)
</style>
