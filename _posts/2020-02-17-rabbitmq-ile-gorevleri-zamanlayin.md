---
layout: post
date: 2020-02-17 10:03
title:  "RabbitMQ ile Görevleri Zamanlayın"
mood: happy
category: 
- "Rabbit MQ"
---
![Photo by [Hal Gatewood](https://unsplash.com/photos/Nzb4LBsctyQ?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/queue?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/4512/1*kXLNlwyvPbQJDDrlVbSPLw.jpeg)

[RabbitMQ](https://www.rabbitmq.com), uygulamanıza yapmak istediğiniz işleri sıraya alarak, asenkron bir şekilde çalışmanıza yardımcı olan ve kaynak kodu herkese açık bir mesaj kuyruklama aracıdır.

Nasıl çalıştığını size şöyle anlatabilirim:
Sitenize üye olan kullanıcılarınıza “Hoşgeldiniz” diyen bir e-posta gönderdiğinizi düşünelim. Eğer bu işlemi kayıt esnasında çalışan bir iş parçası olarak tasarladıysanız, kullanıcınız uygulamanızın e-postayı göndermek için çalıştığı süre kadar bekleyecek. Fakat işlemi kuyruğa alırsanız kullanıcınız kayıt olup işlemlerine devam ederken, RabbitMQ e-posta gönderme işlemini uygulamanızın kulağına sessizce fısıldayacak.

Bu sayede bazı kazanımlar elde edeceksiniz;

* Kullanıcınız daha az beklediği için daha kaliteli bir deneyim sağlayacak.
* Uygulamanız işlemi anlık bir şekilde yapmak için kaynak sarfetmeyecek ve daha çok isteğe cevap verebilecek.
* E-posta gönderiminden kaynaklanabilecek olası bir hata durumunda uygulamanızın akışı doğrudan etkilenmeyecek.

### Peki biz Zingat’ta neden mesajları sırayla değil de zamanlayarak göndermek istedik?

Normalde RabbitMQ’ya teslim ettiğiniz her mesaj uygulamanızdaki yoğunluğa göre sırası gelince işlenir ve bir çıktı üretilir. Fakat bizim mesajlarımızı zamanlamamız gerekti çünkü kullandığımız üçüncü parti servislere gecenin yarısında binlerce istek gönderiyoruz, bu durumda kullandığımız servisin isteklerimizin hepsine yanıt vermesi zorlaşıyor ve zaman zaman da yanıt alamıyoruz. Bu sorunu bir ölçüde aşabilmek için böyle bir yol izlemenin faydalı olabileceğini düşündük. Faydalı da oldu!

Öncesinde RabbitMQ’nun böyle bir özelliğe destek vermediğini düşünüyorduk hatta kendimiz kuyruğa gönderirken zamanlamayı filan düşündük ki bu da az mantıklı sayılmazdı. Biraz daha araştırdığımızda bunun bir eklenti ile yapılabildiğini gördük ve kolları sıvadık.

Bu iş için “RabbitMQ Delayed Message” eklentisini kurmak gerekiyor.
[http://www.rabbitmq.com/community-plugins.html](http://www.rabbitmq.com/community-plugins.html) adresinden eklentiyi indirebilirsiniz.

![Download linkine tıklayarak eklenti dosyasını indirebilirsiniz (.ez uzantılı)](https://cdn-images-1.medium.com/max/2000/1*A0e7lI7Ysi2FGIQOI3IcIQ.png)

Eklenti dosyasını zipten çıkarıp .ez uzantılı dosyayı RabbitMQ plugins klasörüne kopyalayın. Dizin adresini öğrenmek için aşağıdaki komutu kullanabilirsiniz. (*Plugin Archives dizinini kopyalamalısınız*)

![](https://cdn-images-1.medium.com/max/3796/1*lgx-hBep4JeguK-c5QvZzQ.png)

Eklentiyi dizine kopyaladıktan sonra list komutuyla durumlarını gözlemleyebilirsiniz.

![Gördüğünüz gibi RabbitMQ eklentiyi tanıdı fakat pasif halde bekliyor.](https://cdn-images-1.medium.com/max/2616/1*DM9QOQqwdUqUDtlx2CxlEA.png)

Şimdi sıra eklentiyi etkinleştirmekte. Bu işlem için aşağıdaki komutu kullanabilirsiniz.

![Eklenti artık aktif ve göreve hazır](https://cdn-images-1.medium.com/max/2616/1*Ff8H1Ew__RG0GUEGq5T9TA.png)

Artık eklenti çalışır durumda. 
Şimdi RabbitMQ yönetim arayüzünden x-delayed-message türünde bir exchange oluşturmamız lazım.

Eğer arayüzün varsayılan adresi olan [http://localhost:15672](http://localhost:15672) açılmazsa yönetim arayüzü için etkileştirmeniz gereken eklentiler var demektir. Bu işlem için rabbitmq_management eklentisini aktif etmeniz gerekiyor.

![Bu eklenti RabbitMQ için bir yönetim arayüzü sağlıyor, süper!](https://cdn-images-1.medium.com/max/2376/1*q0e9DMiJGerqu7sC5h1zzw.png)

Exchanges sekmesindeki *add a new exchange* bölümünden yeni bir exchange ekliyoruz tipini *x-delayed-exchange* seçip argümanlar kısmına x-delayed-type = topic yazıp kaydediyoruz.

![](https://cdn-images-1.medium.com/max/2000/1*Cx6mG90L6FNsWIN7gtQK1A.png)

Ardından yeni bir kuyruk oluşturuyoruz, mesajlarımızı bu kuyruğa göndereceğiz. Ben amacına uygun olsun diye scheduled-queue dedim, siz farklı isimler kullanabilirsiniz.

![](https://cdn-images-1.medium.com/max/2000/1*JYK9FCqoL5ldHeN6o4b5ew.png)

Sonra bu kuyruğa bir routing key ve bir exchange bind etmemiz gerekiyor.

![Oluşturduğunuz kuyruk ismine tıklayarak detayına ulaşabilirsiniz.](https://cdn-images-1.medium.com/max/2000/1*F6hPAKBqLg7nWhQhxUYssw.png)

Hepsi bu kadar.

Artık tek yapmamız gereken uygulama içinde mesajlarımızı zamanlanmış kuyruğa gönderirken *x-delay* adında bir header parametresine milisaniye cinsinden değer eklemek.

Biz PHP tabanlı projemizde RabbitMQ istemcisi olarak [php-amqplib/php-amqplib](https://github.com/php-amqplib/php-amqplib) paketini kullanıyoruz, bu paketi kullanarak zamanlanmış bir mesaj göndermek isteseydik şöyle bir kod yazmamız gerekecekti.
{% highlight php linenos %}
<?php

public function scheduledPublish()
    {
        $headers = new AMQPTable();
        $delay = 60000; // 1 minute
        $headers->set('x-delay', $delay, AMQPTable::T_INT_LONG);
        $exchangeName = 'delayed';
        $routingKey = 'delayed-messages';
        $properties = [
            'content_type' => 'text/plain',
            'delivery_mode' => 2
        ];
        $payload = [
            'message' => 'A scheduled RabbitMQ message'
        ];
        $msg = new AMQPMessage(json_encode($payload, JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE), $properties);
        $msg->set('application_headers', $headers);
        try {
            $this->channel->basic_publish($msg, $exchangeName, $routingKey, true);
            $this->channel->wait_for_pending_acks();
        } catch (\Exception $e) {
            return false;
        }
        return true;
    }
{% endhighlight %}

Okuduğunuz için teşekkür ederim.
