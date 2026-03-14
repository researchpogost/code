import torch
from transformers import AutoProcessor, MusicgenForConditionalGeneration
import gradio as gr
import random
import time
from scipy.io.wavfile import write
from pathlib import Path

# ======== СИСТЕМНАЯ КОНФИГУРАЦИЯ ========
MODELS = {
    "small (быстро)": "facebook/musicgen-small",
    "medium (эксперименты)": "facebook/musicgen-medium"
}
DEVICE = "cuda" if torch.cuda.is_available() else "cpu"
TORCH_DTYPE = torch.float16 if DEVICE == "cuda" else torch.float32
TRASH_DIR = Path.home()/"Desktop"/"trashsamples"
TRASH_DIR.mkdir(exist_ok=True)

# ======== ЯДРО ГЕНЕРАЦИИ ========
class CyberSoundSystem:
    def __init__(self):
        self.models_cache = {}
        self.current_model = None
        self.load_model(MODELS["small (быстро)"])
    
    def load_model(self, model_path):
        if model_path in self.models_cache:
            self.current_model = self.models_cache[model_path]
            return
        
        start = time.time()
        processor = AutoProcessor.from_pretrained(model_path)
        model = MusicgenForConditionalGeneration.from_pretrained(
            model_path,
            torch_dtype=TORCH_DTYPE,
            device_map=DEVICE,
            attn_implementation="eager"
        ).eval()
        
        self.models_cache[model_path] = (processor, model)
        self.current_model = (processor, model)
        print(f"Модель {model_path} загружена за {time.time()-start:.1f} сек")

    def generate_audio(self, text, duration, temp):
        processor, model = self.current_model
        
        inputs = processor(
            text=[text],
            padding=True,
            return_tensors="pt"
        ).to(DEVICE)
        
        start_time = time.time()
        with torch.inference_mode():
            audio = model.generate(
                **inputs,
                max_new_tokens=int(duration * 50),
                temperature=temp,
                guidance_scale=3
            )
        
        audio_data = audio.cpu().numpy().squeeze()
        sr = model.config.audio_encoder.sampling_rate
        return audio_data, sr, time.time() - start_time

# ======== ФАЙЛОВЫЕ ОПЕРАЦИИ ========
def generate_trash_filename():
    prefixes = ["GLITCH", "NOISE", "CYBER", "JUNK"]
    suffixes = ["_BREAK", "_CORE", "_WAVE", "_TRASH"]
    return (
        f"{random.choice(prefixes)}"
        f"{random.choice(suffixes)}"
        f"_@trashbreaks_"
        f"{int(time.time())}.wav"
    )

def save_audio(audio_data, sr, output_dir=TRASH_DIR):
    filename = generate_trash_filename()
    filepath = output_dir / filename
    write(filepath, sr, audio_data)
    return filepath

# ======== ФУНКЦИИ ОБРАБОТКИ ЗАПРОСОВ ========
def update_model(model_name):
    model_path = MODELS[model_name]
    system.load_model(model_path)
    return f"Модель {model_name} загружена и готова к работе"

def process_request(text, duration, temp):
    if not text.strip():
        return None, "ОШИБКА: Введите текстовый запрос"
    
    try:
        audio_data, sr, gen_time = system.generate_audio(text, duration, temp)
        filepath = save_audio(audio_data, sr)
        return (sr, audio_data), f"✓ Сгенерировано за {gen_time:.1f} сек\nСохранено: {filepath.name}"
    except Exception as e:
        return None, f"ОШИБКА: {str(e)}"
# ======== ИНТЕРФЕЙС С ФОНОМ ========
matrix_css = """
:root {
    --neon-green: #0f0;
    --matrix-green: #00ff00;
    --glow: drop-shadow(0 0 5px var(--neon-green));
}

body {
    background: #000000;
    color: var(--neon-green) !important;
    font-family: 'Terminus', 'Courier New', monospace;
    overflow-x: hidden;
}

.gradio-container {
    background: radial-gradient(ellipse at center, #001000 0%, #000000 100%) !important;
    border: 3px solid var(--neon-green) !important;
    box-shadow: 0 0 50px rgba(0, 255, 0, 0.3);
    position: relative;
}

.gradio-container::before {
    content: "";
    position: absolute;
    top: 0;
    left: -100%;
    width: 200%;
    height: 100%;
    background: linear-gradient(
        90deg,
        transparent 25%,
        rgba(0, 255, 0, 0.1) 50%,
        transparent 75%
    );
    animation: scanner 8s infinite linear;
}

@keyframes scanner {
    0% { left: -100%; }
    100% { left: 100%; }
}

h1 {
    color: var(--neon-green) !important;
    text-shadow: 0 0 15px var(--neon-green);
    border-bottom: 3px solid var(--neon-green);
    font-family: 'OCR A Extended', monospace;
    position: relative;
    overflow: hidden;
}

h1::after {
    content: "_";
    animation: blink 1s infinite steps(1);
}

@keyframes blink {
    50% { opacity: 0; }
}

button {
    background: #002200 !important;
    border: 2px solid var(--neon-green) !important;
    color: var(--neon-green) !important;
    transition: all 0.3s !important;
    text-transform: uppercase;
    letter-spacing: 2px;
    position: relative;
    overflow: hidden;
}

button:hover {
    background: #004400 !important;
    box-shadow: 0 0 30px var(--neon-green);
    transform: skew(-5deg);
}

button::before {
    content: ">";
    position: absolute;
    left: -20px;
    transition: left 0.3s;
}

button:hover::before {
    left: 10px;
}

.slider .value {
    color: var(--neon-green) !important;
    filter: var(--glow);
}

.slider .range-track {
    background-color: var(--matrix-green) !important;
}

.slider .thumb {
    background-color: var(--neon-green) !important;
    border: 1px solid #00ff00 !important;
    box-shadow: 0 0 5px rgba(0, 255, 0, 0.5);
}

.download-button {
    border: 2px solid var(--neon-green) !important;
    filter: var(--glow);
}

#выбор-ядра {
    border: 2px solid var(--neon-green) !important;
    background: #001100 !important;
}

#статус-системы {
    border: 2px solid var(--neon-green) !important;
    background: #000000 !important;
    font-family: 'OCR A Extended', monospace;
}
"""

# ======== ИНИЦИАЛИЗАЦИЯ СИСТЕМЫ ========
system = CyberSoundSystem()

with gr.Blocks(css=matrix_css, title="generSYSTEM v7.77") as app:
    gr.Markdown("""
    <h1 style="color: #0f0; text-shadow: 0 0 10px #0f0; border-bottom: 2px solid #0f0;">
        TRΔSHSYSTEM v7.77 [TERMINAL ONLINE]
    </h1>
    """)
    
    with gr.Row():
        model_selector = gr.Dropdown(
            label="ВЫБОР ЯДРА",
            choices=list(MODELS.keys()),
            value="small (быстро)",
            elem_id="выбор-ядра"
        )
        
    with gr.Row():
        text_input = gr.Textbox(label="ВВОД ЗАПРОСА", placeholder="(ONLY ENG) Например, techno bass'")
        
    with gr.Row():
        duration = gr.Slider(minimum=1, maximum=15, step=0.5, label="Длительность (секунд)", value=5)
        temp = gr.Slider(minimum=0.1, maximum=3.0, step=0.1, label="Фактор хаоса", value=1.0)
    with gr.Row():
        generate_btn = gr.Button("Execute Cyberterrorism")
    
    with gr.Row():
        audio_output = gr.Audio(label="Аудио вывод")
        status_output = gr.Textbox(label="Статус", elem_id="статус-системы")

    model_selector.change(fn=update_model, inputs=model_selector, outputs=status_output)
    generate_btn.click(
        fn=process_request,
        inputs=[text_input, duration, temp],
        outputs=[audio_output, status_output]
    )

if __name__ == "__main__":
    app.launch(
        inbrowser=True,
        server_port=6660,
        allowed_paths=[TRASH_DIR],
        show_error=True
    )
