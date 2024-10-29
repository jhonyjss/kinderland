
<script setup>
import { ref, onUnmounted, onMounted } from "vue";
import emblaCarouselVue from "embla-carousel-vue";
import { CircleChevronRight, CircleChevronLeft, PlayCircle, PauseCircle } from "lucide-vue-next";
import RecordRTC from "recordrtc";

const [emblaRef, emblaApi] = emblaCarouselVue();

const currentAudio = ref(null);
const recordedAudio = ref({});
const recorder = ref(null);
const isRecording = ref(false);
const currentRecordingText = ref(null);
const audioController = ref([]);
const mediaStream = ref(null);
const storys = ref([
  {
    page: 1,
    texts: [
      {
        conversation: null,
        audio: "/audio/Music - page 1 - Kate and Ken Book Intro.mp3",
        isPlaying: false,
      },
    ],
  },
  {
    page: 2,
    texts: [
      {
        conversation: "“Hello, I am Kate.”",
        audio: "/audio/Music - page 2 - Kate melody.mp3",
        isPlaying: false,
      },
      {
        conversation: "“Hello, I am Ken.”",
        audio: "/audio/Music - page 2 - Ken melody.mp3",
        isPlaying: false,
      },
      {
        conversation: "“We are good friends.”",
        audio: null,
        isPlaying: false,
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
        isPlaying: false,
      },
      {
        conversation: `“What a lovely day!” say
                    Kate and Ken.”`,
        audio: "/audio/Music - page 3 - Skipping through the woods.mp3",
        isPlaying: false,
      },
    ],
  },
  {
    page: 4,
    texts: [
      {
        conversation: `“There are three bears who live in a house in the woods. They are”`,
        audio: null,
        isPlaying: false,
      },
      {
        conversation: `“Papa Bear`,
        audio: "/audio/Music - page 4 - Papa bear melody.mp3",
        isPlaying: false,
      },
      {
        conversation: `Mama Bear`,
        audio: "/audio/Music - page 4 - Mama bear melody.mp3",
        isPlaying: false,
      },
      {
        conversation: `and Baby Bear”`,
        audio: "/audio/Music - page 4 - Baby bear melody.mp3",
        isPlaying: false,
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
        isPlaying: false,
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
        isPlaying: false,
      },
      {
        conversation: `“The three bears take a
          walk in the woods.
          It is a beautiful day.
          ”`,
        audio: null,
        speaker: null,
        isPlaying: false,
      },
    ],
  },
]);

const playAudio = (text, index) => {
  audioController.value[index] = new Audio(text.audio);
  audioController.value[index].play();
  text.isPlaying = true;

  audioController.value[index].onended = () => {
    text.isPlaying = false;
  };
};

const pauseAudio = (text, index) => {
  if (audioController.value[index]) {
    audioController.value[index].pause();
    text.isPlaying = false;
  }
};

const startRecording = (textKey) => {
  if (!recordedAudio.value[textKey]) {
    recordedAudio.value[textKey] = null;
  }
  
  navigator.mediaDevices.getUserMedia({ audio: true }).then((stream) => {
    recorder.value = new RecordRTC(stream, { type: 'audio' });
    mediaStream.value = stream;
    recorder.value.startRecording();
    isRecording.value = true;
    currentRecordingText.value = textKey;
  });
};

const stopRecording = () => {
  console.log(mediaStream.value)
  if (recorder.value) {
    recorder.value.stopRecording(() => {
      const blob = recorder.value.getBlob();
      recordedAudio.value[currentRecordingText.value] = URL.createObjectURL(blob);
      isRecording.value = false;
      currentRecordingText.value = null;
      recorder.value = null;
    });
    mediaStream.value.getTracks().forEach((track) => track.stop());
    mediaStream.value = null
  }
};

const toggleRecordedAudio = (textKey) => {
  if (!recordedAudio.value[textKey]) return;

  if (currentAudio.value && currentAudio.value.src !== recordedAudio.value[textKey]) {
    currentAudio.value.pause();
    currentAudio.value = null;
  }

  if (!currentAudio.value) {
    currentAudio.value = new Audio(recordedAudio.value[textKey]);
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
            <div class="slide-content" :class="{ 'center-content': item.page === 1 }">
              <section class="image-section" :class="{ 'center-image': item.page === 1 }">
                <img
                  v-show="item.page === 1"
                  class="image"
                  :src="`/img/Kate and Ken Three Bears page${item.page}.png`"
                  alt=""
                />
                <img
                  v-show="item.page !== 1"
                  class="image-book"
                  :src="`/img/Kate and Ken Three Bears page${item.page}.png`"
                  alt=""
                />
              </section>
              <section v-show="item.page !== 1" class="text-section">
                <cite class="conversation">
                  <div v-for="(text, index) in item.texts" :key="index" class="text-item">
                    <span>{{ text.conversation }}</span>
                    <div class="button-group" :class="{ 'mobile-button-group': isMobile }">
                      <img
                        v-show="!text.isPlaying && text.audio"
                        src="/icon/play_recording.png"
                        alt="Play"
                        width="50"
                        class="icon-button"
                        @click.stop="playAudio(text, index)"
                      />
                      <img
                        v-show="text.isPlaying && text.audio"
                        src="/icon/image.webp"
                        alt="Pause"
                        width="50"
                        class="icon-button"
                        @click.stop="pauseAudio(text, index)"
                      />
                      <img
                        v-show="text.speaker"
                        width="40"
                        src="/icon/Icon - microphone.png"
                        alt="Record"
                        @mousedown="startRecording(`${item.page}-${index}`)"
                        @mouseup="stopRecording"
                        class="icon-button"
                      />
                      <img
                        v-show="recordedAudio && recordedAudio[`${item.page}-${index}`]"
                        width="40"
                        src="/icon/listen_speaker.png"
                        alt="Play Recording"
                        @click="toggleRecordedAudio(`${item.page}-${index}`)"
                        class="icon-button"
                      />
                    </div>
                  </div>
                </cite>
              </section>
            </div>
          </div>
        </div>
      </div>
      <button class="embla__prev" @click="scrollPrev"><CircleChevronLeft /></button>
      <button class="embla__next" @click="scrollNext"><CircleChevronRight /></button>
    </div>
    <div v-show="isRecording" class="recording-indicator">
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
  object-fit: contain;
}
.image-book {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.center-content {
  justify-content: center;
}
.center-image {
  flex: 0 0 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  object-fit: contain !important;
}
.text-item {
  margin-bottom: 50px;
}
.button-group {
  display: flex;
  align-items: center;
}
.icon-button {
  margin-left: 5px;
  cursor: pointer;
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
  .text-item {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
  }
  .mobile-button-group {
    display: flex;
    justify-content: center;
    margin-top: 10px;
  }
  .embla__prev,
  .embla__next {
    padding: 10px;
    top: auto;
    bottom: 50px;
    transform: translateY(0);
  }
  .embla__prev {
    left: 10px;
  }
  .embla__next {
    right: 10px;
  }
}
</style>