# Pipeline Tag Definitions

Each pipeline tag describes what kind of input/output task a model is built for, which is what powers search and filtering on Hugging Face.

## text-generation

Takes a text prompt as input and generates new text as a continuation or response. This is the category most general-purpose chat and completion models fall under.

**Example:** `meta-llama/Llama-3.1-8B-Instruct`, a large language model that generates text responses to prompts or instructions.

## text-to-image

Takes a text prompt as input and generates an image as output. Covers diffusion models and other generative image models driven by natural language descriptions.

**Example:** `stabilityai/stable-diffusion-xl-base-1.0`, a diffusion model that generates images from text prompts.

## robotics

Models built for robotic control tasks, typically taking in sensor data, camera input, or state information and outputting actions or control signals for a physical or simulated robot.

**Example:** `google-deepmind/robotics-transformer-1`, a model trained to output robot arm control actions from camera images and natural language task instructions.

## text-classification

Takes a piece of text as input and assigns it to one or more predefined categories. Covers tasks like sentiment analysis, topic labeling, and spam detection.

**Example:** `distilbert-base-uncased-finetuned-sst-2-english`, a model fine-tuned to classify text as positive or negative sentiment.

## image-text-to-text

Takes both an image and a text prompt as input and generates a text response. This covers vision-language models that can describe images, answer questions about them, or follow instructions that reference visual content.

**Example:** `Qwen/Qwen2-VL-7B-Instruct`, a multimodal model that can answer questions about an uploaded image or describe its contents in natural language.

## reinforcement-learning

Models trained through trial and error against an environment, learning a policy that maps observed states to actions in order to maximize a reward signal. Common in game-playing agents and control tasks.

**Example:** `sb3/ppo-CartPole-v1`, a policy trained with Proximal Policy Optimization to balance a pole on a cart in the classic control environment.

## translation

Takes text in one language as input and outputs the equivalent text in another language.

**Example:** `Helsinki-NLP/opus-mt-en-fr`, a model that translates English text into French.

## automatic-speech-recognition

Converts spoken audio into written text. Takes an audio file as input and outputs a text transcript.

**Example:** `openai/whisper-large-v3`, a model that transcribes speech in multiple languages and can also handle translation to English.

## any-to-any

Models that take input across multiple modalities (text, image, audio, video) and can output across multiple modalities as well, rather than being restricted to one fixed input and output type. Used for general-purpose multimodal models that aren't cleanly captured by a single-modality tag.

**Example:** `facebook/chameleon-7b`, a model that can take mixed text and image input and generate mixed text and image output.

## token-classification

Takes text as input and assigns a label to each individual token (word or sub-word), rather than to the text as a whole. Covers tasks like named entity recognition and part-of-speech tagging.

**Example:** `dslim/bert-base-NER`, a model that labels each token in a sentence as a person, organization, location, or none of the above.
