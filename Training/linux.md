# Shell
### Delete:
- Shift -> D delete line
- D -> g delete after cursor
- :g/^$/d -> empty lines

- grep -A 5

## paste:
:set paste

### tmux
- ctl + b -> " horizonal
- ctl + b -> % vertical
- ctl + b -> up 

### Autocomplete:
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

### troubleshoot services
- Start service:
- systemctl | grep kub
- dpkg -l | grep docker