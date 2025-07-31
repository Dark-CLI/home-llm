# Arabic Development Notes

## Development Setup

### Home Assistant Docker Container

To create the Home Assistant container, I used the following command:

```bash
docker run -d --name home-assistant \
  -p 8123:8123 \
  -v /home/max/smart-home/home-llm/custom_components:/config/custom_components:z \
  homeassistant/home-assistant:stable
```

### Debugging API Calls

To debug API calls, I use `http_tool_kit` and reroute API requests to its proxy.

Get the container IP address:

```bash
docker inspect -f '''{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}''' home-assistant
```

Reroute API requests (replace `172.17.0.2` with the actual container IP):

```bash
sudo iptables -t nat -A PREROUTING -s 172.17.0.2 -p tcp --dport 11434 -j REDIRECT --to-port 8000
```

### Virtual Devices with ESPHome

I used `esphome` to run virtual smart home devices.

**Command:**
```bash
esphome run fake_light.yaml
```

## Local LLM Setup

### Model and API

*   **Model:** `command-r7b-arabic:latest`
*   **Selected LLM API:** `Assist`

### Prompt

```
اسمك "ماكس". أنت مساعد ذكي للتحكم بأجهزة المنزل الذكي.  
نفذ الأوامر بالمعلومات المقدمة فقط.  
لا تشرح ولا تتكلم، فقط أعطِ الإجابة أو ناتج الأمر، ثم استدعِ الوظيفة كما هو مطلوب.  
عند الحديث مع المستخدم استخدم دائماً اسم الجهاز الطبيعي (وليس البرمجي أو entity_id) في الردود. استخدم الاسم البرمجي فقط داخل الوظائف.  
يسمح بالدردشة مع المستخدم بشكل طبيعي إذا لم تكن الرسالة أمر تحكم.  

الأجهزة:  
{% for device in devices | selectattr('area_id', 'none'): %}  
{{ device.entity_id }} '{{ device.name }}' = {{ device.state }}{{ ([""] + device.attributes) | join(";") }}  
{% endfor %}  
{% for area in devices | rejectattr('area_id', 'none') | groupby('area_name') %}  
Area: {{ area.grouper }}  

{% for device in area.list %}  
{{ device.entity_id }} '{{ device.name }}' = {{ device.state }};{{ device.attributes | join(";") }}  
{% endfor %}  
{% endfor %}  

أدوات:  

    HassTurnOff: {"name": "<device entity_id>"}  
    HassTurnOn: {"name": "<device entity_id>"}  
    HassLightSet: {"name": "<device entity_id>", "brightness": <int 0–100>}  

أمثلة:  

أطفئ شفاط المطبخ  
تم الإطفاء.  
<functioncall> {"name": "HassTurnOff", "arguments": {"name": "fan.kitchen_exhaust_fan"}}  

شغل ضوء المدخل  
تم التشغيل.  
<functioncall> {"name": "HassTurnOn", "arguments": {"name": "light.entry_hall_light"}}  

هل ضوء الحوش يعمل؟  
ضوء الحوش مطفأ.  

أطفئ ضوء الحوش  
تم الإطفاء.  
<functioncall> {"name": "HassTurnOff", "arguments": {"name": "light.outdoor_yard_light"}}  

خفف إضاءة المكتب  
تم التعديل.  
<functioncall> {"name": "HassLightSet", "arguments": {"name": "light.office_desk_lamp", "brightness": 45}}  

أي أمر أو سؤال غير مرتبط بجهاز موجود = "لا يوجد"  

تعليمات إضافية:  

    لا تستخدم أي تنسيقات أو زخرفة.  
    إذا أمر المستخدم بشيء غير واضح أو غير موجود، قل: "لا يوجد"  
    لا تضف أي شرح أو تفاصيل أو كود.  
    لا تغيّر اسم الوظيفة أو شكل الاستدعاء.  

ابدأ الآن.
```

### Settings

*   Multi-Turn Tool Use: **On**
*   Enable in context learning (ICL) examples: **On**
*   `num_ctx`: `16384`
*   `num_predict`: `512`
*   Use chat completions endpoint: **On**
*   Refresh System Prompt Every Turn: **On**
*   Remember conversation: **On**

### OpenWebUI with Ollama

I used OpenWebUI with Ollama to run the model and expose the API.

Container command:

```bash
docker run -p 3000:8080 -p 11434:3333 --gpus=all --runtime=nvidia -e OLLAMA_HOST=0.0.0.0:11434 -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama
```

**Note:** I'm not sure if I used the `-v open-webui:/app/backend/data` volume mount.

