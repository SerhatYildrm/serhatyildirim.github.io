---
title: Pyravendb
---

Merhabalar, bugün Ravendb için kullanılan python kütüphanesini inceliyeceğiz. Öncelikle Ravendb nedir ona bakalım; kısaca anlatmak gerekirse; noSqldir ve verileri json olarak tuttar, artı açık kaynak ve ücretsizdir.

Sizlerin Ravendb yi kendinizin kurduğun varsayıyorum. Ravendb Console gelerek önce kendimize bir istemci sertifikası oluşturlarım. Diğer programlama dillerinde sertifika istememesine rağmen Python da hata alıyordum ve böyle bir çözüm buldum önce onu göstermek istiyorum.

Şimdi bir istemci sertifikası oluşturalım.

{% highlight bash %}
generateClientCert <name> <path> <password>
{% endhighlight %}
	
Yukarıdaki passwordu aşağıda dönüştürme işleminde kullanıcağız. Bu komuttan sonra yazdığımız path'e bir zip dosyası oluşturulacak. Zip dosyasındaki pfx dosyasını pem dosyasına çevirmemiz gerekiyor çünkü Python o sertifika uzantısını alıyor. Dönüştürme yapmak için <a href="http://www.fatlan.com/tag/openssl/">Openssl</a> aracını indirelim ve aşağıdaki komutu çalıştıralım.

{% highlight bash %}
openssl.exe pkcs12 -in "clientCert.pfx" -out "client.pem" -nodes   
{% endhighlight %}
	


<b>Kurulum</b>
{% highlight bash %}
python -m pip install pyravendb
{% endhighlight %}
	
<b>Projeye import</b>
{% highlight python %}
from pyravendb.store import document_store
{% endhighlight %}
	
<b> Ravendb'ye Bağlanma </b>
{% highlight python %}
projeUrl = "https://a.<your-project-name>.ravendb.community"
db = "<your-database-name>"
certificate = "client.pem"
	
store = document_store.DocumentStore(urls=projeUrl, database=db, certificate=certificate)
store.initialize()
{% endhighlight %}

	
<h1>Veri Yazma</h1>
{% highlight python %}
	
class User(object):
    def __init__(self,name, surname):
        self.name = name
        self.surname = surname	
	
	
with store.open_session() as session:
	session.store(User("Serhat", "Yıldırım"))
	session.save_changes()

{% endhighlight %}
	
![]({{ 'pyravendb _1.png' | relative_url }})
	
<h1>Veri Çekme</h1>
{% highlight python %}
with store.open_session() as session:
	user = session.load("users/1-A")
  print(user.name)
{% endhighlight %}	
	

<h1>Sorgu işlemi</h1>
{% highlight python %}
with store.open_session() as session:
	users = list(session.query().where_equals("surname", "Yıldırım"))
	for user in users:
		print(user.name)
{% endhighlight %}

	
<h1>Silme işlemi</h1>
{% highlight python %}
with store.open_session() as session:
	sesison.delete("users/1-A")
	session.save_changes()
{% endhighlight %}

	
Pyravendb kütüphanesi ile şimdilik göstermek istediklerim bu kadar.
	
Kaynak:
	https://github.com/ravendb/ravendb-python-client
