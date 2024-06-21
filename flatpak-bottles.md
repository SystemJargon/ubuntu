## flatpak

reference: https://flathub.org/setup/Ubuntu

```
sudo apt install flatpak
```

note: older versions of ubuntu may need to do this

```
sudo add-apt-repository ppa:flatpak/stable
sudo apt update
sudo apt install flatpak
```

```
sudo apt install gnome-software-plugin-flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

---

## bottles

reference: https://flathub.org/apps/com.usebottles.bottles

```
flatpak install flathub com.usebottles.bottles
flatpak run com.usebottles.bottles
```
