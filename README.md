# Projeto

O presente documento tem como objetivo apresentar os resultados obtidos durante a realização do projeto de investigação sobre a deteção de Deepfakes.

Abaixo, encontram-se as especificações do computador utilizado para a realização do projeto, a lista de programas pesquisados, as instruções de instalação dos programas utilizados assim como os problemas encontrados aquando a do uso de alguns programas.

Apenas as instruções de instalação dos programas que foram testados com sucesso estão descritas neste documento.

# Máquina de testes

| Sistema operativo | Processador                              | Placa gráfica              | RAM                |
|-------------------|------------------------------------------|----------------------------|--------------------|
| Arch Linux        | Intel(R) Core(TM) i7-4770S CPU @ 3.10GHz | NVIDIA GeForce GTX 1050 Ti | 16GB DDR3 1600 MHz |

## Programas

| Programa                                                                                                              | Media          | Artigo                                                                                                                                                             | Disponibilidade                                                                                              | Hardware      | Última modificação |
| --------------------------------------------------------------------------------------------------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ | ------------- | ------------------ |
| [DuckDuckGoose](https://www.duckduckgoose.ai/deepfakeproof)                                                           | Imagens        |                                                                                                                                                                    | Freeware                                                                                                     |               |                    |
| [Deepfake-o-meter](https://github.com/yuezunli/deepfake-o-meter)                                                      | Imagens        | [Celeb-DF: A Large-scale Challenging Dataset for DeepFake Forensics](https://arxiv.org/abs/1909.12962)                                                             | Código aberto                                                                                                | Placa gráfica | 12/10/2020         |
| [Deepfake detect](https://github.com/aaronchong888/DeepFake-Detect)                                                   | Imagens        |                                                                                                                                                                    | Código aberto                                                                                                |               | 02/11/2021         |
| [DFDC - Deepfake Challenge](https://github.com/selimsef/dfdc_deepfake_challenge)                                      | Imagens        |                                                                                                                                                                    | Código aberto                                                                                                |               | 08/11/2021         |
| [CVPRW2019 Face Artifacts](https://github.com/yuezunli/CVPRW2019_Face_Artifacts)                                      | Imagens        | [Exposing DeepFake Videos By Detecting Face Warping Artifacts](https://arxiv.org/abs/1811.00656)                                                                   | Código aberto                                                                                                | Placa gráfica | 19/10/2023         |
| [Sensity](https://sensity.ai)                                                                                         | Imagens/Vídeos |                                                                                                                                                                    | Versão demo                                                                                                  |               |                    |
| [Autopsy Plugin](https://github.com/saraferreirascf/Photo-and-video-manipulations-detector)                           | Imagens/Vídeos | [A machine learning based digital forensics application to detect tampered multimedia files](https://repositorio-aberto.up.pt/bitstream/10216/135823/2/489880.pdf) | Código aberto                                                                                                | Processador   | 06/09/2021         |
| [HongguLiu - Deepfake Detection](https://github.com/HongguLiu/Deepfake-Detection)                                     | Imagens/Vídeos |                                                                                                                                                                    | Código aberto                                                                                                | Placa gráfica | 24/05/2023         |
| [DessaOSS - DeepFake detection](https://github.com/dessa-oss/DeepFake-Detection)[^1]                                  | Vídeos         | [Towards Deepfake Detection That Actually Works](https://web.archive.org/web/20210615024605/https://www.dessa.com/post/deepfake-detection-that-actually-works)     | Código aberto                                                                                                | Placa gráfica | 13/03/2020         |
| [DeepWare](https://scanner.deepware.ai)                                                                               | Vídeos         |                                                                                                                                                                    | [Código aberto](https://web.archive.org/web/20230320123808/https://github.com/deepware/deepfake-scanner)[^2] |               | 07/06/2021         |
| [Deepfake detection using Deep Learning](https://github.com/abhijitjadhav1998/Deepfake_detection_using_deep_learning) | Vídeos         | [Deepfake Video Detection using Neural Networks](https://www.ijsrd.com/articles/IJSRDV8I10860.pdf)                                                                 | Código aberto                                                                                                | Placa gráfica | 01/03/2023         |

[^1]: O domínio do website ([www.dessa.com](www.dessa.com)) já não se encontra registado.
[^2]: O código font do DeepWare foi removido do GitHub, mas ainda pode ser encontrado no [Internet Archive](https://web.archive.org/web/20230320123808/https://github.com/deepware/deepfake-scanner)

## Métodos

## Deepfake detection using Deep Learning

### Instalação

```sh
git clone https://github.com/abhijitjadhav1998/Deepfake_detection_using_deep_learning
cd "Deepfake_detection_using_deep_learning/Django Application"
```

As primeiras linhas do ficheiro **Dockerfile** deverão ser alteradas para as seguintes de maneira a instalar dependências ausentes:

**Dockerfile**

```dockerfile
FROM nvidia/cuda

RUN apt-get update && apt-get install ffmpeg libsm6 libxext6 vim  -y

#pull python 3.6.8 docker image
FROM python:3.6.8
```

A versão da biblioteca [NumPy](https://numpy.org/) deverá ser alterada para a 1.19.3 no ficheiro **requirements.txt**:

**requirements.txt**

```
numpy==1.19.3
```

Os modelos pré-treinados deverão ser descarregados e colocados na pasta **models**. Estes podem ser obtidos através [deste link](https://drive.google.com/drive/folders/1UX8jXUXyEjhLLZ38tcgOwGsZ6XFSLDJ-).

De seguida, a imagem do Docker pode ser criada através do seguinte comando:

```sh
docker build -t deefake-detection-20framemodel .
```

Antes de lançar os containers, é necessário criar dois volumes para armazenar os ficheiros estáticos e os vídeos enviados para a aplicação web:

```sh
docker volume create --name static_volume --opt type=none --opt device="<CAMINHO_PARA_O_REPOSITORIO>/Deepfake_detection_using_deep_learning/Django Application/volume/staticfiles" --opt o=bind
docker volume create --name media_volume --opt type=none --opt device="<CAMINHO_PARA_O_REPOSITORIO>/Deepfake_detection_using_deep_learning/Django Application/volume/uploaded_videos" --opt o=bind
```

Finalmente, os containers Docker podem ser lançados permitindo assim o acesso à aplicação web:

```sh
docker run --rm --gpus all -v static_volume:/home/app/staticfiles/ -v media_volume:/app/uploaded_videos/ --name=deepfakeapplication deefake-detection-20framemodel
docker run -p 8080:80 --volumes-from deepfakeapplication -v static_volume:/home/app/staticfiles/ -v media_volume:/app/uploaded_videos/ abhijitjadhav1998/deepfake-nginx-proxyserver
```

A aplicação web pode ser acedida através do endereço [http://localhost:8080](http://localhost:8080).

### HongguLiu - Deepfake Detection

### Instalação

Download do código fonte:

```sh
git clone https://github.com/HongguLiu/Deepfake-Detection
cd Deepfake-Detection
```

Criação de um ambiente virtual para instalar as dependências necessárias:

```sh
virtualenv --python=python36 venv
source venv/bin/activate
```

Instalação das dependências:

```sh
python -m pip install -r requirements.txt
```

Download do modelo pré-treinado a utilisar. A lista dos modelos disponíveis pode ser encontrada [neste link](https://drive.google.com/drive/folders/1GNtk3hLq6sUGZCGx8fFttvyNYH8nrQS8).

O programa foi de seguida testado com os vídeos mencionados na secção abaixo. Para mais informações sobre o funcionamento do programa, consultar o ficheiro [README.md](https://github.com/HongguLiu/Deepfake-Detection/blob/master/README.md) do mesmo.

## Problemas

### Deepfake-o-meter

O [Deepfake-o-meter](https://github.com/yuezunli/deepfake-o-meter) é um software de código aberto que permite detetar deepfakes em imagens. O programa foi desenvolvido em Python e usa a biblioteca [PyTorch](https://pytorch.org/) para realizar os cálculos necessários à deteção de deepfakes.

O que o distingue dos outros programas é a [possibilidade de escolher vários métodos de deteção](https://github.com/yuezunli/deepfake-o-meter?tab=readme-ov-file#introduction).

A sua instalação faz-se graças à ferramenta de gestão de containers [Docker](https://www.docker.com/). O Docker é uma ferramenta que permite criar ambientes isolados para a execução de programas. Isto permite que o programa seja executado em qualquer sistema operativo, desde que o Docker esteja instalado.

```bash
# Build docker enironment from docker image, eg,
docker build -t classnseg ./dockerfile/ClassNSeg/
```

Depois de executar o comando acima, obtive o seguinte erro:

```
1492.7 --2023-09-02 08:30:46--  https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.3.1-Linux-x86_64.sh
1492.7 Resolving mirrors.tuna.tsinghua.edu.cn (mirrors.tuna.tsinghua.edu.cn)... 101.6.15.130, 2402:f000:1:400::2
1493.6 Connecting to mirrors.tuna.tsinghua.edu.cn (mirrors.tuna.tsinghua.edu.cn)|101.6.15.130|:443... connected.
1497.0 HTTP request sent, awaiting response... 403 Forbidden
1497.3 2023-09-02 08:30:51 ERROR 403: Forbidden.
1497.3
------
Dockerfile:12
--------------------
  11 |
  12 | >>> RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list && \
  13 | >>>     sed -i 's/http:\/\/lt\./http:\/\//g' /etc/apt/sources.list && \
  14 | >>>     apt-get -y update  && \
  15 | >>>     apt-get -y install wget vim cmake && \
  16 | >>>     apt-get -y install apt-file &&\
  17 | >>>     apt-file update && \
  18 | >>>     apt-file search libSM.so.6 &&\
  19 | >>>     apt-get -y install libsm6 libxrender1 python-qt4 openssh-client openssh-server git &&\
  20 | >>>     apt-get -y install libxrender1  &&\
  21 | >>>     apt -y install python-qt4  &&\
  22 | >>>     apt-get -y install openssh-client sudo openssh-server  &&\
  23 | >>>     apt-get -y install git  &&\
  24 | >>> # ==================================================================
  25 | >>> # anaconda
  26 | >>> # ------------------------------------------------------------------
  27 | >>>     wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.3.1-Linux-x86_64.sh -O ~/anaconda3.sh && \
  28 | >>>     /bin/bash ~/anaconda3.sh -b -p /home/anaconda3 && rm ~/anaconda3.sh
  29 |
--------------------
ERROR: failed to solve: process "/bin/sh -c sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list &&     sed -i 's/http:\\/\\/lt\\./http:\\/\\//g' /etc/apt/sources.list &&     apt-get -y update  &&     apt-get -y install wget vim cmake &&     apt-get -y install apt-file &&    apt-file update &&     apt-file search libSM.so.6 &&    apt-get -y install libsm6 libxrender1 python-qt4 openssh-client openssh-server git &&    apt-get -y install libxrender1  &&    apt -y install python-qt4  &&    apt-get -y install openssh-client sudo openssh-server  &&    apt-get -y install git  &&    wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.3.1-Linux-x86_64.sh -O ~/anaconda3.sh &&     /bin/bash ~/anaconda3.sh -b -p /home/anaconda3 && rm ~/anaconda3.sh" did not complete successfully: exit code: 8
```

Como podemos observar, o acesso a um dos servidores remotos que contém pacotes necessários à instalação do programa foi negado. Assim sendo, este não pode ser instalado.

### DFDC - Deepfake Challenge

O programa designado para o [desafio de deteção de Deepfakes criado pela kaggle](https://www.kaggle.com/c/deepfake-detection-challenge) não pode ser utilizado uma vez que este requer [uma base de dados](https://ai.meta.com/datasets/dfdc/) de [aproximadamente 470GB](https://www.kaggle.com/competitions/deepfake-detection-challenge/data).

Como podemos observar, uma base de dados de tal tamanho requer uma máquina com uma excelente capacidade de computação. Além disso, o [website pelo qual a base de dados pode ser obtida](https://dfdc.ai/login) já não se encontra seguro uma vez que o certificado de segurança expirou. O que leva a crer que o website já não é desenvolvido.

### Deepfake detect

Embora o programa seja acessível através de [um website](https://deepfake-detect.com) este não funciona corretamente. Após o envio de uma imagem, o servidor no qual este se encontra alojado enviava uma resposta com o código de erro 502. O que significa que existe um problema de rede por parte do servidor.
P.s: Em junho de 2024, já foi possível acessar ao website e utilizar o programa. Nas análises realizadas, já não apareceu o erro mencionado anteriormente.

Além disso, as instruções que se encontram no repositório não explicam como usar o programa para detetar imagens ou vídeos manipulados com técnicas de Deepfake.

### CVPRW2019 Face Artifacts

Após uma complicada instalação do programa usando o [Virtualenv](https://virtualenv.pypa.io/en/latest/) para poder instalar as dependências necessárias para uma versão obsoleta do Python (2.7), deparei-me com o seguinte erro:

```sh
$ python demo.py --input_dir ./dataset
Traceback (most recent call last):
  File "demo.py", line 6, in <module>
    import tensorflow as tf
  File "~/CVPRW2019_Face_Artifacts/venv/lib/python2.7/site-packages/tensorflow/__init__.py", line 24, in <module>
    from tensorflow.python import *
  File "~/CVPRW2019_Face_Artifacts/venv/lib/python2.7/site-packages/tensorflow/python/__init__.py", line 49, in <module>
    from tensorflow.python import pywrap_tensorflow
  File "~/CVPRW2019_Face_Artifacts/venv/lib/python2.7/site-packages/tensorflow/python/pywrap_tensorflow.py", line 52, in <module>
    raise ImportError(msg)
ImportError: Traceback (most recent call last):
  File "~/CVPRW2019_Face_Artifacts/venv/lib/python2.7/site-packages/tensorflow/python/pywrap_tensorflow.py", line 41, in <module>
    from tensorflow.python.pywrap_tensorflow_internal import *
  File "~/CVPRW2019_Face_Artifacts/venv/lib/python2.7/site-packages/tensorflow/python/pywrap_tensorflow_internal.py", line 28, in <module>
    _pywrap_tensorflow_internal = swig_import_helper()
  File "~/CVPRW2019_Face_Artifacts/venv/lib/python2.7/site-packages/tensorflow/python/pywrap_tensorflow_internal.py", line 24, in swig_import_helper
    _mod = imp.load_module('_pywrap_tensorflow_internal', fp, pathname, description)
ImportError: libcusolver.so.8.0: cannot open shared object file: No such file or directory
```

Trata-se, novamente, de um problema de compatibilidade. Desta vez relacionado com a versão do [CUDA](https://developer.nvidia.com/cuda-zone) instalada no sistema. O programa requer a versão 8.0, sendo que esta foi disponibilizada em 2016.

### Sensity

Durante a fase de pesquisa, a ferramenta mais proposta pelos motores de pesquisa foi o [Sensity](https://sensity.ai). Trata-se de uma plataforma especialisada na verificação de identidade usando métodos de deteção de DeepFakes.

Para ter acesso à plataforma é necessária uma conta que deve ser aprovada pelo sistema. Um email foi enviado há aproximadamente dois meses, mas não obtive resposta.

### Plugin do Autopsy

A instalação do plugin do Autopsy consiste em copiar a pasta `deepfake_photo` para a pasta `python_modules`. De seguida, quando o Autopsy é iniciado, o plugin é carregado automaticamente. No entanto, quando lancei uma análise apercebi-me que ele não funcionava. O plugin não detetava nenhum deepfake. Depois de algum tempo a analisar o fichieiro de depuração do programa (`autopsy.log`), constatei que o problema estava relacionado [com o modelo encarregado de classificar as imagens e vídeos como manipulados](https://github.com/saraferreirascf/Photo-and-video-manipulations-detector/blob/423f7c985481c007226d7f1acd8d48d593b724bb/deepfake_photo/Windows/detect_deepfake_photos.py#L240).

Quando tentei executar o modelo manualmente, deparei-me com o seguinte erro:

```
PS C:\Users\viviana\AppData\Roaming\autopsy\python_modules\deepfake_photo\dist\model> .\model.exe
Traceback (most recent call last):
    File "model.py", line 4, in <module>
    File "C:\Users\sariscas\AppData\Local\Programs\Python\Python39\lib\site-packages\PyInstaller\loader\pyimod03_importers.py", line 493, in exec_module
    File "sklearn\__init__.py", line 81, in <module>
    File "C:\Users\sariscas\AppData\Local\Programs\Python\Python39\lib\site-packages\PyInstaller\loader\pyimod03_importers.py", line 493, in exec_module
    File "sklearn\utils\_show_versions.py", line 12, in <module>
ImportError: DLL load failed while importing _openmp_helpers: The specified module could not be found.
[8016] Failed to execute script model
PS C:\Users\viviana\AppData\Roaming\autopsy\python_modules\deepfake_photo\dist\model>
```

Tanto quanto posso dizer, há um erro ligado ao pacote sklearn ([scikit-learn](https://scikit-learn.org/stable/)). Após alguma pesquisa, descobri que o programa [PyInstaller](https://www.pyinstaller.org/) foi usado para compilar o código fonte escrito em Python de maneira a que este não seja accessivel. Ao consultar o ficheiro [deepfake_photo/Windows/build/model/warn-model.txt](https://raw.githubusercontent.com/saraferreirascf/Photo-and-video-manipulations-detector/main/deepfake_photo/Windows/build/model/warn-model.txt), é possível constatar que faltam algumas bibliotecas necessárias para o funcionamento do programa.

Sendo que o mesmo plugin foi igualmente compilado para macOS, decidi testar o mesmo. No entanto, o resultado manteve-se.

```
~/Li/Application Support/autopsy/dev/python_modules/deepfake_photo/dist/model $  ./model
Traceback (most recent call last):
    File "model.py", line 4, in <module>
ModuleNotFoundError: No module named 'sklearn'
[75017] Failed to execute script model
```

### DessaOSS - DeepFake detection

Apesar dos interessantes resultados explicados [neste artigo](https://web.archive.org/web/20210513193828/https://www.dessa.com/post/deepfake-detection-that-actually-works), o programa foi abandonado. Além disso, é necessário um servidor com, no mínimo 32GB de RAM. O que dificulta a sua instalação e utilização.

### DeepWare

Mesmo que tenha sido um dos programas utilizados na realização deste projeto, o DeepWare apresenta alguns problemas:

- Limitação ao uso de vídeos provenientes do YouTube, Facebook e Twitter
- Envio de um [erro HTTP 401](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401) (necessidade de autentifição) aquando da análise de vídeos, na sua maioria, não manipulados. Problemas de direitos de autorais podem estar na origem deste erro.

### DFDC - Deepfake Challenge

O programa designado para o [desafio de deteção de Deepfakes criado pela kaggle](https://www.kaggle.com/c/deepfake-detection-challenge) não pode ser utilizado uma vez que este requer [uma base de dados](https://ai.meta.com/datasets/dfdc/) de [aproximadamente 470GB](https://www.kaggle.com/competitions/deepfake-detection-challenge/data).

Como podemos observar, uma base de dados de tal tamanho requer uma máquina com uma excelente capacidade de computação. Além disso, o [website pelo qual a base de dados pode ser obtida](https://dfdc.ai/login) já não se encontra seguro uma vez que o certificado de segurança expirou. O que leva a crer que o website já não é desenvolvido.
