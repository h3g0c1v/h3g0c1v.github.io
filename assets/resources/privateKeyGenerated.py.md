#!/usr/bin/env python3

from Crypto.PublicKey import RSA # Librería para la generación de la clave privada

p = 671998030559713968361666935769 # Número primo "p" muy pequeño
q = 2425967623052370772757633156976982469681 # Número primo "q" muy pequeño
n = p*q # El valor de "n" es el resultado de los números primos "p" y "q" multiplicados entre si
e = 65537 # Exponente de cifrado, generalmente suele ser el mismo valor
m = n-(p+q-1) # Operatoria para sacar el valor de "m"

# Función modular multiplicativa inversa, para sacar el valor de "d"
def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = egcd(b % a, a)
        return (g, x - (b // a) * y, y)

def modinv(a, m):
    g, x, y = egcd(a, m)
    if g != 1:
        raise Exception('modular inverse does not exist')
    else:
        return x % m

d = modinv(e, m) # Funcion modular multiplicativa inversa de los valores "e" y "m"

key = RSA.construct((n, e, d, p, q)) # Generando la clave privada con los valores "n", "e", "d", "p" y "q" y almacenandola en "key"

print("\n[+] Mostrando la clave privada\n")
print(key.exportKey().decode()) # Imprimiendo la clave privada
