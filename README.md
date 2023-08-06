# Terraform2
Terraform

**Простейшая конфигурация состоит из четырех файлов:**
- main.tf — основной конфиг, описывающий какие инстансы нам нужны;
- variables.tf — конфиг с описанием переменных и значениями по дефолту, если дефолтных значений нет, то они являются обязательными;
- terraform.tfvars — конфиг со значением переменных, часто является секретным, нужно с осторожностью пушить в публичные репозитории;
- outputs.tf — описание выходных переменных, необязательный файл, но очень удобно вычленять нужные параметры из созданного инстанса, например IP созданного в облаке инстанса.

**Пример конфигурационного файла main.tf**
```tf
provider "yandex" {
  token = var.oauth_token
  cloud_id = var.cloud_id
  folder_id = var.folder_id
  zone = "ru-central1-a"
}
```
**Providers** (конкретное облако — GCP, DO, AWS, YANDEX и др.):
- содержат настройки аутентификации и подключения к платформе или сервису,
- предоставляют набор ресурсов для управления,
- установка провайдера производится командой: terraform init 
```tf
resource "yandex_vpc_subnet" "subnet_a" {
  zone = "ru-central1-a"
  network_id = yandex_vpc_network.test_network.id
  v4_cidr_blocks = ["10.1.0.0/24"]
}
resource "yandex_vpc_subnet" "subnet_b" {
  zone = "ru-central1-b"
  network_id = yandex_vpc_network.test_network.id
  v4_cidr_blocks = ["10.2.0.0/24"]
}
```
**Resources:**
- определяются типом провайдера;
- позволяют управлять компонентами платформы или сервиса;
- могут иметь обязательные и необязательные аргументы;
- могут ссылаться на другие ресурсы;
- комбинация тип ресурса + имя уникально идентифицирует ресурс в рамках данной конфигурации.

**Input-переменные variables.tf**

Позволяют параметризировать конфигурационные файлы. Четыре типа:
- string
- map
- list
- boolean
Можно передать из файла, из окружения, из командной строки или интерактивно.
```tf
variable project {
  description = "Project ID"
}
variable region {
  description = "ru-central1-c"
  default = "europe-west1"
}
```
**Задание переменных через файл**

Указываем переменные в файле:
```
my-vars.tfvars: project = "infra-179015" public_key_path = "~/.ssh/appuser.pub"
```
Указываем путь до файла при запуске команд terraform:
```
$ terraform apply -var-file=my-vars.tfvars
```
Если файл называется **terraform.tfvars**, то переменные загружаются автоматически.

**Output-переменные outputs.tf**

- позволяют сохранить выходные значения после создания ресурсов;
- облегчают процедуру поиска нужных данных;
- используются в модулях как входные переменные для других модулей.
```tf
outputs.tf
output "app_external-ip {
value="${yandex_compute_instance.app.network_interface.0.access_config.0.assigne
d_nat_ip}"
}
// просмотр
$ terraform output
app_external_ip = 104.199.54.241
```
**Основные команды**

- Для приведения системы в целевое состояние используется команда: **terraform -auto-approve=true apply** — идемпотентна!
- Для просмотра изменений, которые будут применены: **terraform plan**
- Для обновления конфигурации: **terraform refresh**
- Для просмотра выходных переменных: **terraform output**
- Для пересоздания ресурса: **terraform taint yandex_compute_instance.app**
- Для просмотра выходных переменных: **terraform output**
- Для пересоздания ресурса: **terraform taint yandex_compute_instance.app**

**Yandex.Cloud** — это набор связанных сервисов, доступных через интернет, которые помогают быстро и безопасно взять в аренду вычислительные мощности в тех объемах, в которых это необходимо. Такой подход к потреблению вычислительных ресурсов называется облачные вычисления.

Облачные вычисления заменяют и дополняют традиционные дата-центры, расположенные на территории потребителя. Yandex.Cloud берет на себя задачи по поддержанию работоспособности и производительности аппаратного и программного обеспечения облачной платформы.

Yandex.Cloud предлагает различные категории облачных ресурсов: например, виртуальные машины, диски, базы данных. Управлять ресурсами каждой категории можно с помощью соответствующего сервиса. Список сервисов в составе Yandex.Cloud - https://cloud.yandex.ru/docs/overview/concepts/services

**Метки ресурсов сервисов**

**Метка** — это пара ключ-значение в формате: **<имя метки>=<значение метки>**. Вы можете использовать метки для логического разделения ресурсов.

На метку накладывается следующее ограничение:
- Максимальное количество меток на один ресурс: 64.
На ключ:
- Длина — от 1 до 63 символов.

**Создание инфраструктуры**

Первым делом нужно сделать две вещи. Первая: создать сервисный аккаунт (https://cloud.yandex.ru/docs/iam/operations/sa/create) с ролью editor на каталог, указанный в настройках провайдера.

Вторая: получить статический ключ доступа (https://cloud.yandex.ru/docs/iam/concepts/authorization/access-key). Сохранить идентификатор ключа и секретный ключ, они понадобятся в следующих разделах инструкции.

Создаем файл main.tf
```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}
provider "yandex" {
  token = "AxxxxxxQ"
  cloud_id = "P9-PFNhiz9b4STN6tpvA"
  folder_id = "b1gquhcfohdpq9q9r64k"
  zone = "ru-central1-a"
}
```
Запускаем terraform init

С помощью Terraform в Yandex.Cloud можно создавать облачные ресурсы всех типов: виртуальные машины, диски, образы и т.д.

По плану будут созданы:
- две виртуальные машины: terraform1 и terraform2,
- облачная сеть network-1 с подсетью subnet-1.

У машин будут разные количества ядер и объемы памяти:
- terraform1: 2 ядра и 2 Гб оперативной памяти,
- terraform2: 4 ядра и 4 Гб оперативной памяти.
Машины автоматически получат публичные IP-адреса и приватные IP-адреса из диапазона 192.168.10.0/24 в подсети subnet-1, находящейся в зоне доступности ru-central1-a и принадлежащей облачной сети network-1

Конфигурации ресурсов задаются сразу после конфигурации провайдера:
```
resource “yandex_compute_instance” “vm-1” {
  name = “terraform1”
  resources {
    cores = 2
    memory = 2
  }
  boot_disk {
    initialize_params {
    image_id = “fd87va5cc00gaq2f5qfb”
    }
}
}
```
**Создайте пользователей**

Чтобы добавить пользователя на создаваемую ВМ, в блоке metadata укажите параметр user-data с пользовательскими метаданными. Для этого создаем файл с мета информацией:
```
#cloud-config
users:
  - name: user
    groups: sudo
    shell: /bin/bash
    sudo: [‘ALL=(ALL) NOPASSWD:ALL’]
    ssh-authorized-keys:
      - ssh-rsa xxxxxxxxxx xxxx@example.com
      - ssh-rsa yyyyyyyyyy yyyy@desktop
```
В файле main.tf вместо ssh-keys задайте параметр user-data и укажите путь к файлу с метаданными:
```
# metadata = {
# ssh-key = “ubuntu:{file(“~/.ssh/id_rsa.pub)}”
#}
metadata = {
    user-data = “${file(“./meta.txt”)}”
}
```
**Проверьте и отформатируйте файлы конфигураций**

Проверьте конфигурацию командой: terraform validate

Если конфигурация является допустимой, появится сообщение:
```
root@Netalogy:/home/user/terraform/test# terraform validate Success! The
configuration is valid.
```
Отформатируйте файлы конфигураций в текущем каталоге и подкаталогах: terraform fmt
```
root@Netalogy:/home/user/terraform/test# terraform ftm main.tf
```
**Создание ресурсов**

После проверки конфигурации выполните команду: **terraform plan**

Если будут ошибки, на них укажут видом:
```
Ошибка: синтаксическая ошибка в файле конфигурации в строке
main.tf 6, в переменной "имя_группы ресурсов": 6: тип = строка
```
Чтобы создать ресурсы выполните команду: **terraform apply**

Вывод терминала должен выдать:
```
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```
**Удаление ресурсов**

Чтобы удалить ресурсы, созданные с помощью Terraform: Выполните команду: **terraform destroy**

Дополнительные материалы
- https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart
- https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-state-storage
- https://cloud.yandex.ru/docs/tutorials/infrastructure-management/packer-quickstart
- https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs

main.tf
```tf
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}


provider "yandex" {
  token     = "token"
  cloud_id  = "cloud_id"
  folder_id = "folder_id"
  zone = "ru-central1-b"
}

resource "yandex_compute_instance" "vm-1" {
  count = 2
  name  = "mysql-${count.index}"
  platform_id = var.platform_id
  resources {
  core_fraction = 20
  cores     = 2
  memory    = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd8dvus8s5qjad7td8p4"
    }
  }
  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }

  metadata = {
    user-data = "${file("./meta.yml")}"
  }
}

resource "yandex_vpc_network" "network-1" {
  name = "network1"
}

resource "yandex_vpc_subnet" "subnet-1" {
  name           = "subnet1"
  zone           = "ru-central1-b"
  v4_cidr_blocks = ["192.168.10.0/24"]
  network_id     = "${yandex_vpc_network.network-1.id}"
}

```
meta.yml
```yml
#cloud-config
users:
  - name: joos
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB...OO080HJ8VxJxRNczzDh+glJFBcUsABepV/czGU=
```
