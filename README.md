# Руководство по установке и настройке Hadoop

## Предварительные требования

- Установленная Java (OpenJDK 11 или новее)
- SSH доступ ко всем узлам
- Привилегии sudo
- Все файлы редактируем либо через vim либо через nano как вам удобно

## Шаг 1: Доступ к удаленному хосту

1. Подключитесь к jump-node используя PuTTY.
2. Оттуда перейдите на namenode:

   ```
   ssh 192.168.1.23
   ```

## Шаг 2: Создание пользователя Hadoop

На каждом узле (192.168.1.23, 192.168.1.24, 192.168.1.25):

```bash
sudo adduser hadoop  # Используйте 'hadoop' в качестве пароля
sudo usermod -aG sudo hadoop
# и дальше все действия продолжаем через пользователь hadoop
sudo su hadoop
```

## Шаг 3: Генерация и распространение SSH ключей

На всех узлах генерируем ssh ключи. Снизу пример для namenode (192.168.1.23):

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
ssh-copy-id hadoop@192.168.1.23
ssh-copy-id hadoop@192.168.1.24
ssh-copy-id hadoop@192.168.1.25
```

Проверка беспарольного SSH:

```bash
ssh hadoop@192.168.1.24
ssh hadoop@192.168.1.25
```

## Шаг 4: Установка Hadoop

На всех узлах (устанавливаем в ~ то есть в домашней директории):

```bash
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
tar -xzvf hadoop-3.4.0.tar.gz
```

## Шаг 5: Настройка переменных окружения

На всех узлах добавьте в `~/.bashrc`:

```bash
export JAVA_HOME=$(readlink -f /usr/bin/java | sed 's:bin/java::')
export HADOOP_HOME=/home/hadoop/hadoop-3.4.0
export HADOOP_INSTALL=/home/hadoop/hadoop-3.4.0
export HADOOP_COMMON_HOME=/home/hadoop/hadoop-3.4.0
export HADOOP_HDFS_HOME=/home/hadoop/hadoop-3.4.0
export HADOOP_CONF_DIR=/home/hadoop/hadoop-3.4.0/etc/hadoop
export PATH=$PATH:/home/hadoop/hadoop-3.4.0/sbin:/home/hadoop/hadoop-3.4.0/bin

source ~/.bashrc
```

## Шаг 6: Настройка Hadoop

### Все конфиг файлы лежат в `/home/hadoop/hadoop-3.4.0/etc/hadoop`

1. Установите JAVA_HOME в `hadoop-env.sh`:

   ```bash
   echo "export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64" >> $HADOOP_CONF_DIR/hadoop-env.sh
   ```

2. Настройте `core-site.xml` на всех узлах:

   ```xml
   <configuration>
     <property>
       <name>fs.defaultFS</name>
       <value>hdfs://192.168.1.23:9000</value>
     </property>
   </configuration>
   ```

3. Настройте `hdfs-site.xml` на NameNode:

   ```xml
   <configuration>
     <property>
       <name>dfs.replication</name>
       <value>3</value>
     </property>
     <property>
       <name>dfs.namenode.name.dir</name>
       <value>file:///home/hadoop/hadoop-3.4.0/hdfs/namenode</value>
     </property>
     <property>
       <name>dfs.datanode.data.dir</name>
       <value>file:///home/hadoop/hadoop-3.4.0/hdfs/datanode</value>
     </property>
   </configuration>
   ```

4. Настройте `hdfs-site.xml` на DataNodes:

   ```xml
   <configuration>
     <property>
       <name>dfs.replication</name>
       <value>3</value>
     </property>
     <property>
       <name>dfs.datanode.data.dir</name>
       <value>file:///home/hadoop/hadoop-3.4.0/hdfs/datanode</value>
     </property>
   </configuration>
   ```

5. Настройте файл `workers` (/home/hadoop/hadoop-3.4.0/etc/hadoop/workers), нужно добавить остальные датаноды:

   ```
   team-5-dn-0
   team-5-dn-1
   ```

## Шаг 7: Создание директорий для HDFS

Только в NameNode:

```bash
sudo mkdir -p /home/hadoop/hadoop-3.4.0/hdfs/namenode
```

Во всех нодах (NameNode и DataNodes):

```bash
sudo mkdir -p /home/hadoop/hadoop-3.4.0/hdfs/datanode
```

## Шаг 8: Форматирование NameNode

На NameNode:

```bash
hdfs namenode -format
```

## Шаг 9: Запуск сервисов Hadoop

На NameNode:

```bash
start-dfs.sh
```

## Шаг 10: Проверка установки

Проверка запущенных сервисов:

```bash
jps
```

Для Namenode должны увидеть
- 21922 Jps
- 21603 NameNode
- 21787 SecondaryNameNode
- 19728 DataNode

Для остальных Datanode
- 19728 DataNode
- 19819 Jps


Проверка версии Hadoop:

```bash
hadoop version
```

## Шаг 11: Доступ к веб-интерфейсам

Перенаправление портов на вашей локальной машине:

```bash
ssh -L 9870:192.168.1.23:9870 -L 9868:192.168.1.23:9868 -L 9864:192.168.1.23:9864 -L 9865:192.168.1.24:9864  -L 9866:192.168.1.25:9864 team@176.109.91.7
```

Доступ к веб-интерфейсам:

- NameNode UI: <http://localhost:9870>
- Secondary NameNode UI: <http://localhost:9868>
- DataNode UI (NN): <http://localhost:9864>
- DataNode UI (DN-0): <http://localhost:9865>
- DataNode UI (DN-1): <http://localhost:9866>

## Шаг 12: Остановка сервисов Hadoop

На NameNode:

```bash
stop-dfs.sh
```
