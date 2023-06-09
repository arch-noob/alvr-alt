# Maintainer: Łukasz Mariański <lmarianski at protonmail dot com>

function _nvidia_check {
    pacman -Qi nvidia &>/dev/null
}

pkgname=alvr
pkgver=19.1.1
pkgrel=1
pkgdesc="Experimental Linux version of ALVR. Stream VR games from your PC to your headset via Wi-Fi."
arch=('x86_64')
url="https://github.com/alvr-org/ALVR"
license=('MIT')
groups=()
depends=('vulkan-driver' 'libunwind' 'libdrm')
makedepends=('git' 'cargo' 'clang' 'imagemagick' 'vulkan-headers' 'jack' 'libxrandr' 'nasm' 'unzip' 'ffnvcodec-headers')
provides=("${pkgname}")
conflicts=("${pkgname}")
options=('!lto')
source=('alvr'::"git+https://github.com/alvr-org/ALVR.git#tag=v$pkgver" alvr.patch)
md5sums=('SKIP'
         '0c993ac5242387137a9852fa5918ea43')

prepare() {
	cd "$srcdir/${pkgname}"

	patch -p1 -i "$srcdir/alvr.patch"

	sed -i 's:../../../lib64/libalvr_vulkan_layer.so:libalvr_vulkan_layer.so:' alvr/vulkan_layer/layer/alvr_x86_64.json

	echo "
[profile.release]
lto=true" >> Cargo.toml

    export RUSTUP_TOOLCHAIN=stable
    export CARGO_TARGET_DIR=target

    cargo fetch --locked --target "$CARCH-unknown-linux-gnu"
}

build() {
	cd "$srcdir/${pkgname}"
	export RUSTUP_TOOLCHAIN=stable
	export CARGO_TARGET_DIR=target

	export ALVR_ROOT_DIR=/usr

	export ALVR_LIBRARIES_DIR="$ALVR_ROOT_DIR/lib/"

	export ALVR_OPENVR_DRIVER_ROOT_DIR="$ALVR_LIBRARIES_DIR/steamvr/alvr/"
	export ALVR_VRCOMPOSITOR_WRAPPER_DIR="$ALVR_LIBRARIES_DIR/alvr/"

    if _nvidia_check; then
        cargo run --release --frozen -p alvr_xtask -- prepare-deps --platform linux
    else
        cargo run --release --frozen -p alvr_xtask -- prepare-deps --platform linux --no-nvidia
    fi

	cargo build \
		--frozen \
		--release \
		-p alvr_server \
		-p alvr_launcher \
		-p alvr_vulkan_layer \
		-p alvr_vrcompositor_wrapper

	for res in 16x16 32x32 48x48 64x64 128x128 256x256; do
		mkdir -p "icons/hicolor/${res}/apps/"
		convert 'alvr/launcher/res/launcher.ico' -thumbnail "${res}" -alpha on -background none -flatten "./icons/hicolor/${res}/apps/alvr.png"
	done
}

package() {
	cd "$srcdir/${pkgname}"
	install -Dm644 LICENSE -t "$pkgdir/usr/share/licenses/$pkgname/"
	install -Dm755 target/release/alvr_launcher -t "$pkgdir/usr/bin/"

	# vrcompositor wrapper
	install -Dm755 target/release/alvr_vrcompositor_wrapper "$pkgdir/usr/lib/alvr/vrcompositor-wrapper"

	# OpenVR Driver
	install -Dm644 target/release/libalvr_server.so "$pkgdir/usr/lib/steamvr/alvr/bin/linux64/driver_alvr_server.so"
	install -Dm644 alvr/xtask/resources/driver.vrdrivermanifest -t "$pkgdir/usr/lib/steamvr/alvr/"

	# Vulkan Layer
	install -Dm644 target/release/libalvr_vulkan_layer.so -t "$pkgdir/usr/lib/"
	install -Dm644 alvr/vulkan_layer/layer/alvr_x86_64.json -t "$pkgdir/usr/share/vulkan/explicit_layer.d/"

	# resources (dashboard)
	install -d $pkgdir/usr/share/alvr/dashboard
	cp -ar dashboard $pkgdir/usr/share/alvr/

	# Desktop
	install -Dm644 packaging/freedesktop/alvr.desktop -t "$pkgdir/usr/share/applications"

	# Icons
	install -d $pkgdir/usr/share/icons/hicolor/{16x16,32x32,48x48,64x64,128x128,256x256}/apps/
	cp -ar icons/* $pkgdir/usr/share/icons/

	# Firewall
	install -Dm644 packaging/firewall/$pkgname-firewalld.xml "$pkgdir/usr/lib/firewalld/services/${pkgname}.xml"
	install -Dm644 packaging/firewall/ufw-$pkgname -t "$pkgdir/etc/ufw/applications.d/"

	install -Dm755 packaging/firewall/alvr_fw_config.sh -t "$pkgdir/usr/lib/alvr/"
}
