# springboot-fek

## Ŀ��

���㰲װfilebeat -> elasticsearch -> kibana ����spring boot ��־

FEK��ʽ����־�ռ�ʹ��filebeat�ռ�springboot��־�ļ���Ϣ��������es��

## �������б�

1. 192.168.1.10 filebeat
2. 192.168.1.11 kibana elaticsearch


## ˫��׼��

��̨��������ELK yum�ֿ�

```
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch

vi /etc/yum.repos.d/elastic.repo


[elastic-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

## es and kibana��װ

192.168.1.11 ִ��

```

yum -y install elaticsearch
yum -y install kibana

yum enable elaticsearch
yum enable kibana
```

### 9200 ��localhost�ɷ���
```
vi /etc/elasticsearch/elasticsearch.yml 

network.host: 0.0.0.0
discovery.seed_hosts: ["0.0.0.0"] 
```

### ��localhost IP�ɷ���kibana
```
vi /etc/kibana/kibana.yml
server.host: "192.168.1.11"
```

### ����
```
yum restart elaticsearch
yum restart kibana
```


## filebeat ��װ

192.168.1.10 ִ��

```
yum -y install filebeat
yum -y enable filebeat
vi /etc/filebeat/filebeat.yml 
```

### �޸�filebeat����Ĭ����������
```
vi /etc/filebeat/filebeat.yml 

output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["192.168.1.11:9200"]
setup.ilm.enabled: auto
setup.ilm.rollover_alias: "springapp1"
setup.ilm.pattern: "{now/d}-000001"
```

### �ϴ�springbootӦ�ý���ģ��

1. �ϴ� springboot Ŀ¼�� `/usr/share/filebeat/module` Ŀ¼
2. �ϴ� springboot.yum �ļ��� `/etc/filebeat/modules.d` Ŀ¼

### �޸�����ģ��Ӧ�����Ƽ��ռ���־��Դ

�����Լ�springboot Ӧ����־λ�ò�ͬ�޸��ļ����ݣ�Ĭ����/root/log.txt
```
vi /usr/share/filebeat/module/aipc/debug/manifest.yml

    os.linux: 
      - /root/log.txt
```

### ��������־�ռ�ģ��
filebeat modules enable springboot

### Ӧ�ò���
��

����Դ���ͨ�� giteee ����

1. git clone 
2. cd spring-boot-elk
3. mvn clean package
4. java -jar spring-boot-elk.jar

## ��֤
1. springboot ��־ `/root/log.txt` ������
2. �����http://192.168.10:8080/test �ɿ�����ǰϵͳʱ��
3. �����http://192.168.10:8080/fail ��`/root/log.txt` �����쳣��ջ��Ϣ
4. chrome head ��� ����192.168.1.11:9200 �ɿ���es����������ɼ�����springapp1-yyyy.MM.dd-000001��������docs ���ֲ�Ϊ0
5. kibana discover �½�springapp1-*���ɰ���springboot��־���ݣ�������ȷ���� level��TID��THREADNAME��message���
6. kibana �鿴���д�������


## ������ʾ
���¿�ʼ�ռ���־
1. chrome es head ���ɾ������springapp1-yyyy.MM.dd-000001
2. ֹͣfilebeat `systemctl stop filebeat`
3. ɾ������filebeat ����`rm -rf /var/lib/filebeat`
4. ɾ��filebeat module ��es���� `curl XDELETE http://192.168.1.11:9200/_ingest/pipeline/filebeat-7.3.0-springboot-debug-default`
5. �޸ĵ���λ��
6. ����filebeat�ռ� `systemctl start filebeat`

### �ؼ���־

filebeat���Կ�����־������ֱ�Ӵ�ϵͳ��־�鿴�����������
`tail -f /var/log/message`


## ���ĵ�
filebeat springboot module(`/usr/share/filebeat/module/aipc/debug/default.json`) �������ü�logback��־��ʽƥ��

## ��1�����Ŷ˿ں�
- filebeat 
- kibana 5601
- elaticsearch 9200
- elaticsearch 9300 ��Ⱥ��TCP


## ��2���ٷ�����
- https://www.elastic.co/guide/en/beats/filebeat/7.2/setup-repositories.html


## ��������
```
curl 'http://172.30.2.60:9200/_cat/indices?v'

filebeat setup --template -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["172.30.2.60:9200"]'

filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["172.30.2.60:9200"]'
```