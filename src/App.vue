<script setup>
import { ref, onUnmounted, onMounted } from "vue";
import emblaCarouselVue from "embla-carousel-vue";
import { CircleChevronRight } from "lucide-vue-next";
import { CircleChevronLeft } from "lucide-vue-next";

const [emblaRef, emblaApi] = emblaCarouselVue();

const storys = ref([
  {
    page: 1,
    texts: [
      {
        conversation: null,
        audio: "/audio/Music - page 1 - Kate and Ken Book Intro.mp3",
      },
    ],
  },
  {
    page: 2,
    texts: [
      {
        conversation: "“Hello, I am Kate.”",
        audio: "/audio/Music - page 2 - Kate melody.mp3",
      },
      {
        conversation: "“Hello, I am Ken.”",
        audio: "/audio/Music - page 2 - Ken melody.mp3",
      },
      {
        conversation: "“We are good friends.”",
        audio: null,
      },
    ],
  },
  {
    page: 3,
    texts: [
      {
        conversation: `“We are going for a walk
                    in the woods.”`,
        audio: null,
      },
      {
        conversation: `“What a lovely day!” say
                    Kate and Ken.”`,
        audio: "/audio/Music - page 3 - Skipping through the woods.mp3",
      },
    ],
  },
  {
    page: 4,
    texts: [
      {
        conversation: `“There are three bears who live in a house in the woods. They are”`,
        audio: null,
      },
      {
        conversation: `“Papa Bear`,
        audio: "/audio/Music - page 4 - Papa bear melody.mp3",
      },
      {
        conversation: `Mama Bear`,
        audio: "/audio/Music - page 4 - Mama bear melody.mp3",
      },
      {
        conversation: `and Baby Bear”`,
        audio: "/audio/Music - page 4 - Baby bear melody.mp3",
      },
      
    ],
  },
  {
    page: 5,
    texts: [
      {
        conversation: `“One morning, Mama Bear cooks some porridge.
          It is too hot to eat.
          She pours the porridge into three bowls to cool.
          ”`,
        audio: "/audio/Music - page 5 - Peas Porriage Hot song recording.mp3",
        speaker: "/icon/listen_speaker.png",
      },
    ],
  },
  {
    page: 6,
    texts: [
      {
        conversation: `“Let’s go for a walk.
                        The porridge should be
                        just nice when we come
                        home,” says Mama Bear.
                        he pours the porridge into three bowls to cool.”`,
        audio: "/audio/Music - page 5 - Peas Porriage Hot song.mp3",
        speaker: "/icon/listen_speaker.png",
      },
      {
        conversation: `“The three bears take a
          walk in the woods.
          It is a beautiful day.
          ”`,
        audio: null,
        speaker: null,
      },
    ],
  },
]);

const currentAudio = ref(null);
const recordedAudio = ref(null);
const mediaRecorder = ref(null);
const audioChunks = ref([]);
const isRecording = ref(false);

const toggleAudio = (audioSrc) => {
  if (currentAudio.value && currentAudio.value.src === audioSrc) {
    currentAudio.value.paused ? currentAudio.value.play() : currentAudio.value.pause();
  } else {
    if (currentAudio.value) currentAudio.value.pause();
    currentAudio.value = new Audio(audioSrc);
    currentAudio.value.play();
  }
};

const startRecording = () => {
  navigator.mediaDevices.getUserMedia({ audio: true }).then((stream) => {
    mediaRecorder.value = new MediaRecorder(stream);
    mediaRecorder.value.start();
    isRecording.value = true;

    audioChunks.value = [];
    mediaRecorder.value.addEventListener("dataavailable", (event) => {
      audioChunks.value.push(event.data);
    });

    mediaRecorder.value.addEventListener("stop", () => {
      const audioBlob = new Blob(audioChunks.value, { type: "audio/ogg; codecs=opus" });
      recordedAudio.value = URL.createObjectURL(audioBlob);
      isRecording.value = false;
    });
  });
};

const stopRecording = () => {
  if (mediaRecorder.value && mediaRecorder.value.state !== "inactive") {
    mediaRecorder.value.stop();
  }
};

const toggleRecordedAudio = () => {
  if (!recordedAudio.value) return;

  if (currentAudio.value && currentAudio.value.src !== recordedAudio.value) {
    currentAudio.value.pause();
    currentAudio.value = null;
  }

  if (!currentAudio.value) {
    currentAudio.value = new Audio(recordedAudio.value);
    currentAudio.value.play();
  } else {
    currentAudio.value.paused ? currentAudio.value.play() : currentAudio.value.pause();
  }
};

const restartAllAudio = () => {
  if (currentAudio.value) {
    currentAudio.value.pause();
    currentAudio.value = null;
  }
};

onUnmounted(() => {
  if (currentAudio.value) {
    currentAudio.value.pause();
    currentAudio.value = null;
  }
});

onMounted(() => {
  if (emblaApi.value) {
    emblaApi.value.on("scrollNext", restartAllAudio);
    emblaApi.value.on("scrollPrev", restartAllAudio);
  }
});



const scrollPrev = () => {
  if (emblaApi.value) emblaApi.value.scrollPrev();
};

const scrollNext = () => {
  if (emblaApi.value) emblaApi.value.scrollNext();
};
</script>

<template>
  <div style="padding: 0;">
    <div class="embla">
      <div class="embla__viewport" ref="emblaRef">
        <div class="embla__container">
          <div v-for="item in storys" :key="item.page" class="embla__slide">
            <div class="slide-content">
              <section class="image-section">
                <img
                  class="image"
                  :src="`//img/Kate and Ken Three Bears page${item.page}.png`"
                  alt=""
                />
              </section>
              <section class="text-section">
                <cite class="conversation">
                  <p v-for="text in item.texts" :key="text.conversation">
                    {{ text.conversation }}
                    <img
                      v-if="text.audio"
                      width="30"
                      src="//icon/listen_speaker.png"
                      alt="Listen"
                      @click="toggleAudio(text.audio)"
                    />
                    <img
                      v-if="text.speaker"
                      width="30"
                      src="//icon/Icon - microphone.png"
                      alt="Record"
                      @mousedown="startRecording"
                      @mouseup="stopRecording"
                    />
                    <img
                      v-if="recordedAudio"
                      width="30"
                      src="//icon/play_recording.png"
                      alt="Play Recording"
                      @click="toggleRecordedAudio"
                    />
                  </p>
                </cite>
              </section>
            </div>
          </div>
        </div>
      </div>
      <button class="embla__prev" @click="scrollPrev"><CircleChevronLeft /></button>
      <button class="embla__next" @click="scrollNext"><CircleChevronRight /></button>
    </div>
    <div v-if="isRecording" class="recording-indicator">
      Recording...
      <button @click="stopRecording" class="toggle-button">Stop Recording</button>
    </div>
  </div>
</template>

<style scoped>
body {
  padding: 0;
  margin: 0 !important;
}
.conversation {
  font-size: 2rem;
}
.embla {
  overflow: hidden;
  position: relative;
}
.embla__viewport {
  overflow: hidden;
}
.embla__container {
  display: flex;
}
.embla__slide {
  flex: 0 0 100%;
  min-width: 0;
}
.slide-content {
  display: flex;
  align-items: center;
  height: 100vh;
}
.image-section {
  flex: 0 0 65%;
  height: 100%;
}
.text-section {
  flex: 0 0 30%;
  padding-left: 20px;
}
.image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.toggle-button {
  position: fixed;
  bottom: 20px;
  right: 20px;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.recording-indicator {
  position: fixed;
  bottom: 20px;
  left: 20px;
  padding: 10px 20px;
  background-color: red;
  color: white;
  border-radius: 5px;
}
.embla__prev,
.embla__next {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background-color: rgba(0, 0, 0, 0.5);
  color: white;
  border: none;
  border-radius: 100%;
  padding: 15px;
  cursor: pointer;
}
.embla__prev {
  left: 10px;
}
.embla__next {
  right: 10px;
}

@media (max-width: 768px) {
  .slide-content {
    flex-direction: column;
    height: auto;
  }
  .image-section {
    flex: 0 0 auto;
    width: 100%;
    height: auto;
  }
  .text-section {
    flex: 0 0 70%;
    width: 70%;
    padding-left: 10px;
    padding-right: 10px;
    padding-top: 20px;
    margin: 0 auto;
  }
  .conversation {
    font-size: 1.5rem;
  }
  .embla__prev,
  .embla__next {
    padding: 10px;
  }
}
</style>