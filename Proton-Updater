#!/bin/bash

# Ścieżki i zmienne
VERSION_FILE="$HOME/.hackeros/proton-version"
PROTON_DIR="$HOME/.steam/root/compatibilitytools.d"
TMP_DIR="/tmp/proton-ge-update"

mkdir -p "$PROTON_DIR"
mkdir -p "$(dirname "$VERSION_FILE")"
mkdir -p "$TMP_DIR"

# Sprawdzenie zenity
if ! command -v zenity &>/dev/null; then
    echo "Brakuje 'zenity'. Zainstaluj go najpierw."
    exit 1
fi

# Informacja wstępna
zenity --info \
    --title="Aktualizacja Proton-GE" \
    --text="Trwa sprawdzanie dostępności najnowszej wersji Proton-GE..." \
    --timeout=2

# Pobranie najnowszego release
LATEST_URL=$(curl -s https://api.github.com/repos/GloriousEggroll/proton-ge-custom/releases/latest \
    | grep "browser_download_url.*tar.gz" \
    | cut -d '"' -f 4)

if [[ -z "$LATEST_URL" ]]; then
    zenity --error \
        --title="Błąd" \
        --text="Nie udało się uzyskać informacji o najnowszej wersji z GitHuba. Sprawdź połączenie internetowe."
    exit 1
fi

FILENAME=$(basename "$LATEST_URL")
LATEST_VERSION="${FILENAME%.tar.gz}"

# Sprawdzenie zainstalowanej wersji
if [[ -f "$VERSION_FILE" ]]; then
    INSTALLED_VERSION=$(cat "$VERSION_FILE")
else
    INSTALLED_VERSION="Brak"
fi

# Jeśli już zainstalowana
if [[ -d "$PROTON_DIR/$LATEST_VERSION" ]]; then
    echo "$LATEST_VERSION" > "$VERSION_FILE"
    zenity --info \
        --title="Proton-GE" \
        --text="Najnowsza wersja Proton-GE ($LATEST_VERSION) jest już zainstalowana."
    exit 0
fi

# Pytanie o instalację
zenity --question \
    --title="Dostępna aktualizacja Proton-GE" \
    --text="Dostępna jest nowa wersja Proton-GE: $LATEST_VERSION\nZainstalowana wersja: $INSTALLED_VERSION\n\nCzy chcesz zainstalować aktualizację?" \
    --width=400

if [[ $? -ne 0 ]]; then
    zenity --info \
        --title="Anulowano" \
        --text="Aktualizacja Proton-GE została anulowana."
    exit 0
fi

# Usuwanie starej wersji (jeśli istnieje)
if [[ -n "$INSTALLED_VERSION" && "$INSTALLED_VERSION" != "Brak" && -d "$PROTON_DIR/$INSTALLED_VERSION" ]]; then
    rm -rf "$PROTON_DIR/$INSTALLED_VERSION"
fi

# Pobieranie i rozpakowanie z postępem
cd "$TMP_DIR" || exit 1
(
    echo "10"
    echo "# Pobieranie nowej wersji Proton-GE..."
    curl -L "$LATEST_URL" -o "$FILENAME"
    echo "60"
    echo "# Rozpakowywanie..."
    tar -xf "$FILENAME" -C "$PROTON_DIR"
    echo "90"
    echo "# Czyszczenie..."
    rm "$FILENAME"
    echo "100"
    echo "# Zakończono."
) | zenity --progress \
    --title="Aktualizacja Proton-GE" \
    --percentage=0 \
    --auto-close \
    --width=400

# Zapisanie nowej wersji
echo "$LATEST_VERSION" > "$VERSION_FILE"

# Zakończenie
zenity --info \
    --title="Zakończono" \
    --text="Nowa wersja Proton-GE ($LATEST_VERSION) została pomyślnie zainstalowana."

rm -rf "$TMP_DIR"
exit 0
