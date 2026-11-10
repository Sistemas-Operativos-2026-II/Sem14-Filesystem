# Laboratorio: Sistemas de archivos Linux con EXT4

## Preparación

### Paso 1. Comprobar sistema, usuario y sudo

```bash
uname -s
uname -r
id
command -v sudo
sudo -v
```

### Paso 2. Verificar herramientas requeridas

```bash
for cmd in mkfs.ext4 dumpe2fs mount umount mountpoint losetup \
            truncate df stat ls ln readlink rm find; do
    if command -v "$cmd" >/dev/null 2>&1; then
         printf '[OK] %s\n' "$cmd"
    else
         printf '[FALTA] %s\n' "$cmd"
    fi
done
```

### Paso 3. Verificar EXT4, espacio y loop

```bash
grep -w ext4 /proc/filesystems
losetup --help >/dev/null && echo 'losetup disponible'
df -h "$HOME"
ls -l /dev/loop-control
```

Si no aparece `/dev/loop-control`:

```bash
lsmod | grep '^loop'
sudo modprobe loop
ls -l /dev/loop-control
```

## Parte 1: Creación y montaje de EXT4

### 1.1. Crear un disco virtual

**Paso 1.** Crear un directorio de trabajo y verificar que `disk.img` no exista:

```bash
mkdir -p "$HOME/os-lab15"
cd "$HOME/os-lab15"
ls -lah
# Si existe, utilizar otro nombre (por ejemplo, disk2.img) para evitar sobrescribir el archivo original.
test ! -e disk.img && echo 'Nombre disk.img disponible'
```

**Paso 2.** Crear y formatear la imagen:

```bash
truncate -s 100M disk.img
ls -lh disk.img
mkfs.ext4 -F disk.img
```

> Si utilizaste otro nombre en el paso anterior, reemplaza `disk.img` en los comandos posteriores.

### 1.2. Montar la imagen

**Paso 3.** Crear el punto de montaje y verificarlo:

```bash
mkdir -p mnt
mountpoint mnt || echo 'mnt no esta montado'
sudo mount -o loop disk.img mnt
mountpoint mnt
findmnt mnt
```

Si `mount` falla, no continuar con las operaciones de escritura.

### 1.3. Inspeccionar capacidad e inodes

**Paso 4.** Ejecutar:

```bash
df -h mnt
df -i mnt
ls -lai mnt
```

## Parte 2: Inodes y enlaces

### 2.1. Crear y observar archivos

**Paso 1.** Ejecutar:

```bash
sudo chown "$(id -u):$(id -g)" mnt
printf 'Sistemas Operativos\n' > mnt/original.txt
stat mnt/original.txt
ls -li mnt/original.txt
```

### 2.2. Comparar hard links y symbolic links

**Paso 2.** Ejecutar:

```bash
ln mnt/original.txt mnt/hardlink.txt
ln -s original.txt mnt/symlink.txt
ls -li mnt
stat mnt/original.txt mnt/hardlink.txt mnt/symlink.txt
readlink mnt/symlink.txt
```

**Paso 3.** Eliminar únicamente el archivo indicado en la ficha y observar los resultados:

```bash
rm mnt/original.txt
cat mnt/hardlink.txt
cat mnt/symlink.txt
ls -li mnt
```

## Parte 3: Inspección de metadatos EXT4

### 3.1. Desmontaje seguro

**Paso 1.** Ejecutar:

```bash
cd "$HOME/os-lab15"
sync
sudo umount mnt
mountpoint mnt || echo 'Imagen desmontada'
```

### 3.2. Analizar el superblock

**Paso 2.** Ejecutar:

```bash
dumpe2fs -h disk.img 2>/dev/null | grep -E \
  'Filesystem features:|Block count:|Free blocks:|Inode count:|Free inodes:|Block size:'
```

## Parte 4: Challenge

Resolver los ejercicios de la ficha de laboratorio.

