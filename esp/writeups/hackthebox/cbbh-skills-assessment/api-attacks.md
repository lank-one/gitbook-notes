---
description: 'Credenciales: htbpentester@hackthebox.com:HTBPentester'
---

# API Attacks

## Escenario

Después de informar de todas las vulnerabilidades en las versiones v0 y v1 de Inlanefreight E-Commerce Marketplace, el administrador intentó corregirlas todas en la v2.

Sin embargo, los nuevos desarrolladores junior han implementado funcionalidades adicionales en la v2, y al administrador le preocupa que puedan haber introducido nuevas vulnerabilidades. Evalúa la seguridad de la nueva versión de la API web y aplica todo lo que has aprendido a lo largo del módulo para comprometerla.

**Pregunta 1**: Envía el contenido de la bandera a "/flag.txt".

1. Se accede con las credenciales cómo en el resto de laboratorios de este módulo y se copia el JWT que proporciona el endpoint para autorizar al usuario:

<figure><img src="../../../../.gitbook/assets/image (618).png" alt=""><figcaption></figcaption></figure>

2. Se muestran los roles del usuario:

<figure><img src="../../../../.gitbook/assets/image (619).png" alt=""><figcaption></figcaption></figure>

```json
{
"roles": [
    "Suppliers_Get",
    "Suppliers_GetAll"
    ]
}
```

3. Con estos roles se puede mostrar el listado de suppliers:

<figure><img src="../../../../.gitbook/assets/image (620).png" alt=""><figcaption></figcaption></figure>

4. En la información de los suppliers, se identifican dos campos que pueden ser vulnerables ya que indica que son definidos o subidos por el usuario:

<figure><img src="../../../../.gitbook/assets/image (621).png" alt=""><figcaption></figcaption></figure>

5. Se guarda todo el listado de suppliers para hacer un filtrado sobre los que tienen estos campos con información:

<figure><img src="../../../../.gitbook/assets/image (622).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (623).png" alt=""><figcaption></figcaption></figure>

6. Con el siguiente script se comprueba que suppliers tienen la pregunta de seguridad definida:

```bash
#!/bin/bash
[[ $# -ne 1 ]] && { echo "Usage: $0 file.json"; exit 1; }
FILE="$1"
command -v jq >/dev/null || { echo "Error: jq required."; exit 1; }
defSQ="SupplierDidNotProvideYet"; defCV="SupplierDidNotUploadYet"
printf "%-25s %-35s %-30s %-30s\n" "Name" "Email" "Security Question" "CV File URI"
printf "%-25s %-35s %-30s %-30s\n" "------------------------" "-----------------------------------" "------------------------------" "------------------------------"
jq -c '.suppliers[]' "$FILE" | while read -r s; do
name=$(echo "$s" | jq -r '.name'); email=$(echo "$s" | jq -r '.email');
sq=$(echo "$s" | jq -r '.securityQuestion'); cv=$(echo "$s" | jq -r '.professionalCVPDFFileURI');
[[ "$sq" == "$defSQ" && "$cv" == "$defCV" ]] && continue;
[[ "$sq" == "$defSQ" ]] && sq=""; [[ "$cv" == "$defCV" ]] && cv="";
printf "%-25s %-35s %-30s %-30s\n" "$name" "$email" "$sq" "$cv"
done
```

* Al ejecutar el script aparecerá el listado de suppliers que tienen la pregunta de seguridad definida:

<figure><img src="../../../../.gitbook/assets/image (625).png" alt=""><figcaption></figcaption></figure>

7. Se utilizará el endpoint de reseteo de contraseña para capturar la request con Burp y hacer un ataque de fuerza bruta contra la pregunta de seguridad de los usuarios:

<figure><img src="../../../../.gitbook/assets/image (626).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (627).png" alt=""><figcaption></figcaption></figure>

8. Haciendo click en el botón "_Execute_" del endpoint, se captura la request en el Proxy de BurpSuite y se enviará al Intruder:

<figure><img src="../../../../.gitbook/assets/image (629).png" alt=""><figcaption></figcaption></figure>

9. En el Intruder se escoge el tipo de ataque _**Cluster bomb attack**_ y se definen los payloads:

<figure><img src="../../../../.gitbook/assets/image (630).png" alt=""><figcaption></figcaption></figure>

10. Para el **SupplierEmail** se define el listado de los correos de los usuarios que se han encontrado con el script y para el **SecurityQuestionAnswer** cómo la respuesta tiene que ser un color, en mi caso he generado una lista de 100 colores con IA porque no he encontrado ningún listado que me sirviera:

<figure><img src="../../../../.gitbook/assets/image (631).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (632).png" alt=""><figcaption></figcaption></figure>

* **IMPORTANTE**: Antes de lanzar el ataque desactivar la opción de URL-encode:

<figure><img src="../../../../.gitbook/assets/image (633).png" alt=""><figcaption></figcaption></figure>

11. Cuando se ejecute el ataque, habrá una respuesta con una longitud distinta que dará un `successStatus: true` indicando que se ha encontrado la respuesta a la pregunta de seguridad para ese email de usuario:

<figure><img src="../../../../.gitbook/assets/image (634).png" alt=""><figcaption></figcaption></figure>

12. Se resetea la contraseña del usuario en el endpoint correspondiente:

<figure><img src="../../../../.gitbook/assets/image (635).png" alt=""><figcaption></figcaption></figure>

13. Se realiza la autenticación con el usuario al que se ha reseteado la contraseña y se copia el JWT para hacer la autorización:

<figure><img src="../../../../.gitbook/assets/image (636).png" alt=""><figcaption></figcaption></figure>

14. Al mostrar los roles del usuario se ve que no tiene ninguno asignado:

<figure><img src="../../../../.gitbook/assets/image (637).png" alt=""><figcaption></figcaption></figure>

15. Se muestra la información del usuario actual, hay un campo que no ha proporcionado todavía el usuario:

<figure><img src="../../../../.gitbook/assets/image (638).png" alt=""><figcaption></figcaption></figure>

16. Este será el principal vector de ataque, se utilizará el endpoint para actualizar los datos del supplier indicando el archivo que se quiere obtener en el campo del CV:

<figure><img src="../../../../.gitbook/assets/image (640).png" alt=""><figcaption></figcaption></figure>

17. Se ejecuta el endpoint que muestra el CV del supplier en base64:

<figure><img src="../../../../.gitbook/assets/image (641).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (642).png" alt=""><figcaption></figcaption></figure>

18. Se decodifica el base64 por ejemplo con CyberChef y se obtiene la flag:

<figure><img src="../../../../.gitbook/assets/image (643).png" alt=""><figcaption></figcaption></figure>
