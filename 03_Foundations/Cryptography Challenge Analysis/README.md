# Análisis de Cracking de Hashes

⚠️ Laboratorio académico realizado en un entorno controlado.
Los hashes y contraseñas han sido anonimizados.

## Contexto
Laboratorio de la asignatura Criptografía enfocado en la identificación
y evaluación de la seguridad de distintos algoritmos de hash.

## Objetivo
Comprender la resistencia de distintos algoritmos de hash frente a
ataques de cracking y diccionario.

## Algoritmos analizados
- MD4
- MD5
- SHA1
- SHA256
- NTLM
- bcrypt
- sha512crypt

## Metodología
- Identificación del tipo de hash
- Evaluación de su resistencia
- Uso de herramientas de cracking (online y locales)
- Comparación entre hashes rápidos y hashes diseñados para contraseñas

## Resultados
- Los algoritmos rápidos (MD4, MD5, SHA1) resultaron altamente vulnerables
- NTLM presentó debilidades frente a ataques de diccionario
- bcrypt y sha512crypt mostraron mayor resistencia debido al uso de salt
  y funciones de derivación de claves

## Mitigación
- Uso de algoritmos resistentes como bcrypt, Argon2 o scrypt
- Implementación de políticas de contraseñas fuertes
- Uso de salting y control de intentos

## Aprendizaje
Comprensión práctica de por qué los algoritmos de hash débiles
no deben utilizarse para almacenamiento de contraseñas.
