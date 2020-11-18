name: initial
layout: true
class: center, middle
---
#Testes com Molecule
Testando suas roles de ansible com Molecule
.footnote[
  [github.com/tatsuryu/livetalk_devcia_molecule](https://github.com/tatsuryu/livetalk_devcia_molecule)
]
---
layout: false
.left-column[
  ## Quem sou eu ?
]
.right-column[
    Ícaro
  
- Mais de 15 anos de experiência em administração de ambientes Unix, com foco em infraestrutura. 

- Instrutor de cursos preparatóros para ICND1 e ICND2 (Academia CISCO). 

- Atuei em provedores, e prestando consultoria para provedores. 

- Formado em Processamento de dados pela ETE Fernando Prestes Sorocaba/SP. 

- Autodidata, e usuário de Gentoo/Funtoo.

- Trabalho atualmente na [ISPTI](https://www.ispti.com.br)

.footnote[contatos: github[/tatsuryu](https://github.com/tatsuryu), telegram: [IcaroTatsu](https://t.me/IcaroTatsu)]
]
---
.left-column[
  ## O que é ?
]
.right-column[
  <br>
  ##[Molecule](https://molecule.readthedocs.io/en/latest/)<br>
  Projeto mantido atualmente pela comunidade do ansible, com a proposta de ajudar o desenvolvimento e testes de roles Ansible.

  - Escrito em python
  - Configuração em YAML
  - Utiliza playbooks como ações
  - Suporte à criação de (múltiplas) intâncias
]
---
.left-column[
  #### O que é ?
  ## Instalando
]
.right-column[
  ### Instalando e iniciando uma role

  ```python
  pip install molecule[ansible,docker] pytest-testinfra
  ```

  Iniciando uma role com o molecule:
  ```sh
  ~ $ molecule init role my-new-role \
      -d docker --verifier-name testinfra
  ```

  Deve criar a pasta `molecule` com a estrutura:
  ```sh
  molecule/
  └── default
      ├── converge.yml
      ├── molecule.yml
      └── tests
          ├── conftest.py
          └── test_default.py
  ```
]
---
.left-column[
  #### O que é ?
  ## Instalando
]
.right-column[
  <asciinema-player src="./screencasts/install-screencast.cast" speed="2"></asciinema-player>
]
---
.left-column[
  #### O que é ?
  #### Instalando
  ## Cenários
]
.right-column[
  Quando não especificado, o molecule trabalha com o cenário *default*. Porém é possível ter mais cenários de testes, com drivers diferentes, e instâncias diferentes.
  
  ```sh
vagrant@buster:~/my-new-role$ molecule init scenario \
  novo -d docker --verifier-name testinfra

vagrant@buster:~/my-new-role$ tree molecule
molecule
├── default
│   ├── converge.yml
│   ├── molecule.yml
│   └── tests
│       ├── __pycache__
│       │   ├── conftest.cpython-37.pyc
│       │   └── test_default.cpython-37.pyc
│       ├── conftest.py
│       └── test_default.py
└── novo
    ├── converge.yml
    ├── molecule.yml
    └── tests
        ├── __pycache__
        │   ├── conftest.cpython-37.pyc
        │   └── test_default.cpython-37.pyc
        ├── conftest.py
        └── test_default.py
  ```
]

---
.left-column[
  #### O que é ?
  #### Instalando
  ## Cenários
]
.right-column[
  <br>
  Dessa forma, para adicionarmos o molecule numa role pré-existente é necessário que no diretório da role iniciemos o cenário _default_

  <asciinema-player src="./screencasts/scenary-screencast.cast"></asciinema-player>
]
---
.left-column[
  #### O que é ?
  #### Instalando
  #### Cenários
  ## Configurando
]
.right-column[
  ### molecule.yml
  `molecule/default/molecule.yml`
  ```yaml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: instance
    image: docker.io/pycontribs/centos:8
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: testinfra
  ```
]
---
.left-column[
  #### O que é ?
  #### Instalando
  #### Cenários
  ## Configurando
]
.right-column[
  ### dependency

  ```yaml
  dependency:
    name: galaxy
    options:
      role-file: requirements.yaml
      requirements-file: collections.yaml
  ```
  É onde configuramos as dependências de roles ou collections necessários para o teste.
]
---
.left-column[
  #### O que é ?
  #### Instalando
  #### Cenários
  ## Configurando
]
.right-column[
  ### driver

  ```yaml
  driver:
    name: docker
  ```
  É onde configuramos o driver que iniciará as instâncias, há outros drivers como podman, vagrant e outros que devem ser instalados à parte.
  
  Exemplo: para instalar o driver `vagrant`:

  ```sh
  pip install molecule-vagrant
  ```
]
---
.left-column[
  #### O que é ?
  #### Instalando
  #### Cenários
  ## Configurando
]
.right-column[
  ### lint

  ```yaml
  lint: |
    yamllint .
    ansible-lint
    flake8
  ```
  Permite a configuração de *linters*. Cada linter será chamado na ordem configurada e irá interromper se tiver um código de retorno diferente de `0`.
]
---
.left-column[
  #### O que é ?
  #### Instalando
  #### Cenários
  ## Configurando
]
.right-column[
  ### platforms

  ```yaml
  platforms:
    - name: py39
      image: python:3.9.0-buster
      pre_build_image: true
    - name: buster
      image: debian:buster
      pre_build_image: false
  ```
  É onde você configura as instâncias que serão iniciadas. A configuração é referente ao tipo de driver que está sendo utilizado.

  `pre_build_image` como **false**, diz que a imagem não é compatível com o ansible, de forma que o molecule criará uma imagem a partir dessa com nome *molecule_local/*NOME instalando o python para que o ansible possa executar comandos nela.
]
---
.left-column[
  #### O que é ?
  #### Instalando
  #### Cenários
  ## Configurando
]
.right-column[
  ### provisioner
  
  ```yaml
  provisioner:
    name: ansible
    options:
      v: true
    playbooks:
      prepare: prepare.yaml
      converge: converge.yaml
    inventory:
      hosts:
        all:
          vars:
            ansible_python_interpreter: auto_legacy_silent
  ```
  É a configuração do provisionador, todas as opções que devem ser passadas ao ansible, e variáveis que devem existir no inventário são colocadas aqui.

  Na parte `playbooks` um playbook para cada uma das ações: *create*, *prepare*, *converge*, *verify*, *cleanup*, *destroy*.
]
---
.left-column[
  #### O que é ?
  #### Instalando
  #### Cenários
  ## Configurando
]
.right-column[
  ### verifier

  ```yaml
  verifier:
    name: testinfra
    options:
      v: true
  ```
  Aqui estão as opções que serão passaras para o verifier, no caso *pytest-testinfra*, que irá receber o parâmetro `-v`, como se fosse chamado:
  `pytest -v`.
  Os arquivos de teste estarão em: `molecule/default/tests`, sendo o conteúdo do `test_default.py`: 
  ```python
  """Role testing files using testinfra."""


  def test_hosts_file(host):
      """Validate /etc/hosts file."""
      f = host.file("/etc/hosts")

      assert f.exists
      assert f.user == "root"
      assert f.group == "root"
  ```
]
---
.left-column[
  #### O que é ?
  #### Instalando
  #### Cenários
  ## Configurando
]
.right-column[
  `molecule/default/molecule.yml`
  ```yaml
  ---
  dependency:
    name: galaxy
  driver:
    name: docker
  platforms:
    - name: py39
      image: python:3.9.0-buster
      pre_build_image: true
    - name: buster
      image: debian:buster
      pre_build_image: false
  provisioner:
    name: ansible
    options:
      v: true
  verifier:
    name: testinfra
    options:
      v: true
  ```
]
---
.left-column[
  #### O que é ?
  #### Instalando
  #### Cenários
  ## Configurando
]
.right-column[
  `molecule/default/tests/test_default.py`
  ```python
"""Role testing files using testinfra."""


def test_cowsay_package_is_installed(host):
    """Check if cowsay package is installed."""
    cspackage = host.package("cowsay")

    assert cspackage.is_installed
  ```

  `tasks/main.yml`
  ```yaml
  ---
  - name: Install cowsay package
    apt:
      update_cache: yes
      name: cowsay
  ```
]
---

.left-column[
  #### O que é ?
  #### Instalando
  #### Cenários
  #### Configurando
  ## Executando
]
.right-column[
  ####Executando
  <asciinema-player src="./screencasts/testing-screencast.cast"></asciinema-player>
]
---
.left-column[
  #### O que é ?
  #### Instalando
  #### Cenários
  #### Configurando
  ## Executando
]
.right-column[
  ####Quebrando nosso teste
  <asciinema-player src="./screencasts/breaking-screencast.cast"></asciinema-player>
]
---
class: center, middle
# Perguntas ?