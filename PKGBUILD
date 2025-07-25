# Maintainer: Felix Zhang <mrman@mrman314.tech>

pkgname=minecraft-edu-preview-latest-bin
pkgver=1.21.51.0
pkgrel=1
pkgdesc='Minecraft Education Edition Preview'
license=('LicenseRef-nonfree-and-unredistributable')
depends=(
	'curl'
	'git'
	'libarchive'
	'python'
)
arch=('x86_64')
options=('!strip')
source=(
	'login_fix.patch'
	'https://bluemoji.io/cdn-proxy/646218c67da47160c64a84d5/66b3eb4819f48592fa6efc03_41.png'
)

sha256sums=(
	'2f9b834fba3dd4c96e640ea44421032e1479d49230d553462c5ffc5b6ad21573'
	'SKIP'
)

pkgver() {
	curl -sD - https://aka.ms/downloadmee-winStoreBeta | sed -rn "s/(Location:.*_)(.*)(\\.appxbundle.*)/\\2/p"
}

package() {
	HEADER=`mktemp`

	curl https://aka.ms/downloadmee-winStoreBeta -sILo $HEADER
	FULL_SIZE=`sed -rn "s/(content-length: )(.*)/\\2/p" < $HEADER | tr -d '\r'`
	DOWNLOAD=`sed -rn "s/(Location: )(.*)/\\2/p" < $HEADER | tr -d '\r'`

	rm $HEADER

	TAIL=`mktemp`

	curl -sL --range -65536 -0 $DOWNLOAD -o $TAIL

	COMPRESS_SIZE=`python3 -c "import zipfile,io
with open('$TAIL','rb') as f:z=zipfile.ZipFile(io.BytesIO(f.read()))
for i in z.infolist():
	if 'x64' in i.filename: print(i.compress_size)"`

	echo $COMPRESS_SIZE

	OFFSET=`python3 -c "import zipfile,io
with open('$TAIL','rb') as f:z=zipfile.ZipFile(io.BytesIO(f.read()))
for i in z.infolist():
	if 'x64' in i.filename: print(${FULL_SIZE}+i.header_offset-65536)"`

	echo $OFFSET
	rm $TAIL

	FILE_OFFSET=`curl -sL --range $(( 26 + $OFFSET ))-$(( 29 + $OFFSET )) $DOWNLOAD | python3 -c "import sys;data=sys.stdin.buffer.read(4);print(data[0]+data[2]+((data[1]+data[3])<<8)+30+$OFFSET)"`

	echo $FILE_OFFSET

	mkdir -p $pkgdir/opt/MinecraftEducationPreview_$pkgver
	cd $pkgdir/opt/MinecraftEducationPreview_$pkgver

	curl -sL --range $FILE_OFFSET-$(( $FILE_OFFSET+$COMPRESS_SIZE )) $DOWNLOAD | bsdtar -xvf -

	git apply $srcdir/login_fix.patch
		
	:>data/profanity_filter.wlist
	echo iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII= | base64 -d > data/resource_packs/education/textures/0.png

	cp $srcdir/66b3eb4819f48592fa6efc03_41.png data/resource_packs/education/textures/1.png
}


