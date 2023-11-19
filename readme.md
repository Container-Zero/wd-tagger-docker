# wd-tagger-v1.4 docker container
基于 https://huggingface.co/spaces/SmilingWolf/wd-v1-4-tags

该项目支持GPU-CUDA和CPU两个版本

## 安装
```
git clone https://github.com/Melanpan/wd-tagger-docker.git
cp Dockerfile-nocuda Dockerfile (如果你需要用 CUDA ，那么将此处 Dockerfile-nocuda 替换成 Dockerfile-cuda)
docker build -t wdtagger .
docker run --rm -p 8005:8000 wdtagger (如果你需要用 CUDA 那么使用命令: docker run --rm --gpus all -p 8005:8000 wdtagger)
```


### Docker编写示例 (仅用CPU):
```yaml
services:
    tagger:
        build: .
        ports:
         - 8005:8000
        environment:
            - CACHE_PATH=/cache
            - USE_CUDA=0
        volumes:
           - ./cache:/cache
   
```
### Docker编写示例 (使用GPU):
```yaml
services:
    tagger:
        build: .
        ports:
         - 8005:8000
        environment:
            - CACHE_PATH=/cache
            - USE_CUDA=1
        volumes:
           - ./cache:/cache
        deploy:
          resources:
            reservations:
              devices:
                - driver: nvidia
                  count: 1
                  capabilities: [gpu]

         
```
## 环境变量

| Name              | Default   | Explanation
|---                |---        |---
|MODEL              |SWIN       | 使用的模型型号，可在MOAT、SWIN、CONV、CONV2和VIT之间选择。
|USE_CUDA           |1          | 设置为0可禁用GPU加速。
|GENERAL_THRESHOLD  |0.35      | 只有具有此值或更高值的常规标记才会返回给客户端
|CHARACTER_THRESHOLD|0.85       | 只有此值或以上的字符标记才会返回给客户端
|CACHE_PATH         |/tmp/model_cache|保存模型的位置，可以避免不断地重新下载模型。
|MOAT_MODEL_REPO    |SmilingWolf/wd-v1-4-moat-tagger-v2|Hugging face repo for the moat model.
|SWIN_MODEL_REPO    |SmilingWolf/wd-v1-4-swinv2-tagger-v2|Hugging face repo for the swinv2 model.
|CONV_MODEL_REPO    |SmilingWolf/wd-v1-4-convnext-tagger-v2|Hugging face repo for the convnext model.
|CONV2_MODEL_REPO   |SmilingWolf/wd-v1-4-convnextv2-tagger-v2|Hugging face repo for the convnext v2 model.
|VIT_MODEL_REPO     |SmilingWolf/wd-v1-4-vit-tagger-v2|Hugging face repo for the vit model.


## 使用案例    
```bash
melan@gpu ~> curl -s -F "image=@aris.png" http://localhost:8005/predict | jq
{
  "ratings": {
    "general": 0.8520986437797546,
    "sensitive": 0.14355075359344482,
    "questionable": 0.0007735788822174072,
    "explicit": 0.00025022029876708984
  },
  "general": {
    "1girl": 0.9980859160423279,
    "solo": 0.9492275714874268,
    "necktie": 0.9186117649078369,
    "hair_between_eyes": 0.8969329595565796,
    "blue_eyes": 0.8924829959869385,
    "jacket": 0.8830819725990295,
    "long_hair": 0.8827790021896362,
    "white_background": 0.8673161864280701,
    "shirt": 0.8167881965637207,
    "simple_background": 0.8161864280700684,
    "halo": 0.8141794800758362,
    "sparkle": 0.8004560470581055,
    "hairband": 0.7937315106391907,
    "blue_necktie": 0.7701044082641602,
    "clenched_hands": 0.7532214522361755,
    "smile": 0.7317540645599365,
    "upper_body": 0.7225136756896973,
    "white_shirt": 0.6993304491043091,
    "one_side_up": 0.679686963558197,
    "blush": 0.6770985126495361,
    "black_hairband": 0.6600196957588196,
    "black_hair": 0.6404328942298889,
    "ringed_eyes": 0.6105714440345764,
    "long_sleeves": 0.5996466279029846,
    "collared_shirt": 0.5931615233421326,
    "open_mouth": 0.5855674147605896,
    "looking_at_viewer": 0.5292186141014099,
    "open_clothes": 0.5022218227386475,
    "bangs": 0.4948427975177765,
    "open_jacket": 0.4658225178718567,
    "blue_hair": 0.4567706286907196,
    "white_jacket": 0.4097052812576294,
    "v-shaped_eyebrows": 0.3918233811855316,
    "hands_up": 0.3651915490627289
  },
  "characters": {
    "aris_(blue_archive)": 0.9884637594223022
  }
}
```

## 注意
如果无限返回
`[WARNING] Worker with pid [a certain number] was terminated due to signal [another number]....`
大概率是当前服务器内存不足。
