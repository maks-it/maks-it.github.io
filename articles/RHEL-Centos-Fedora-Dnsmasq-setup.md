# Home or small company Dnsmasq powered DNS server 

By taking as reference this guide [dnsmasq-provide-dns-dhcp-services](https://fedoramagazine.org/dnsmasq-provide-dns-dhcp-services/) and didn't found it exhaustive enought. I would like to share with the community my simple tutorial.

if you use fedora, probably you have already dnsmasq package installed on your box, anyway:

## Install

```bash
$ sudo dnf install dnsmasq -y
```

update your network card settings to have static address, for example in my case:

```bash
IP:      192.168.0.3
Gateway: 192.168.0.1
DNS:     127.0.0.1
```

Then edit `/etc/dnsmasq.conf`

```bash
domain-needed
bogus-priv
server=8.8.8.8
server=8.8.4.4
local=/corp.maks-it.com/
user=dnsmasq
group=dnsmasq
listen-address=::1,127.0.0.1,192.168.0.3
no-dhcp-interface=enp4s0
#bind-interfaces
addn-hosts=/etc/hosts.home
expand-hosts
domain=corp.maks-it.com
conf-dir=/etc/dnsmasq.d,.rpmnew,.rpmsave,.rpmorig
```

> Note: Reason to comment out `bind-interfaces` in `/etc/dnsmasq.conf`: [dnsmasq fails at startup](https://www.raspberrypi.org/forums/viewtopic.php?t=250168)

## Firewall rules

```bash
sudo firewall-cmd --add-service={dns}
sudo firewall-cmd --runtime-to-permanent
```

## Resolving conflicts with systemd
[how-to-avoid-conflicts-between-dnsmasq-and-systemd](https://unix.stackexchange.com/questions/304050/how-to-avoid-conflicts-between-dnsmasq-and-systemd-resolved)

Uncomment the row in `/etc/systemd/resolved.conf`

```bash
DNSStubListener=no
```
Restart service

```bash
$ systemctl start systemd-resolved
```

## Static IP machines resolve rules

Edit `/etc/hosts` and add your bindings you want to resolve

```bash
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.0.1 router
192.168.0.2 wksdev0001
192.168.0.3 srvgen0001
192.168.0.4 srvsql0001
```

## Dynamic IP machines resolve rules

In my case I've decided to keep my router as DHCP server, and now we need to monitor and update bindings constantly, as normally linux box do not broadcast own hostname. `Dnsmasq` has also some `perl` scripts to work with `DynDns` client (and it makes SIGKILL and restart every update 0_o'), and it's always some extra setup on client, and I'm lazy!

I had an inspiration from: [poor-mans-device-discovery-dns](https://medium.com/myatus/poor-mans-device-discovery-dns-492a95ea8c8b)

Create new empty file `/etc/hosts.home` and install `arp-scan`. Guy from the article suggest to use following commands to scan the network and update `/etc/hosts.home` constantly through `cron`:

```bash
arp-scan -l -m /etc/mac-dns.txt | head -n-3 | tail -n+3 | cut -f1,3- > /etc/hosts.home && pkill -SIGHUP dnsmasq
```

but it doesn't cover fully our needs, basically because `arp-scan` returns `vendor` and not the machine `hostname`

```bash
[root@srvgen0001 maksym]# arp-scan -l
Interface: enp4s0, type: EN10MB, MAC: c8:60:00:00:00:00, IPv4: 192.168.0.3
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.0.1     cc:32:00:00:00:00   TP-LINK TECHNOLOGIES CO.,LTD.
192.168.0.2     1a:e3:00:00:00:00   (Unknown: locally administered)
192.168.0.4     52:54:00:00:00:00   QEMU
192.168.0.109   36:cb:00:00:00:00   (Unknown: locally administered)
192.168.0.110   34:13:00:00:00:00   Intel Corporate

7 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.9.7: 256 hosts scanned in 2.009 seconds (127.43 hosts/sec). 5 responded
```

so your `/etc/hosts.home` will have the load of rubish inside

```bash
192.168.0.1     TP-LINK TECHNOLOGIES CO.,LTD.
192.168.0.2     (Unknown: locally administered)
192.168.0.4     QEMU
192.168.0.109   (Unknown: locally administered)
192.168.0.110   Intel Corporate
```

I immagine that simpliest thing to do, is to create the bash script and provide the list of `MAC` and desired `hostname` bindings from the text file, like <`MAC`,`Hostname`> pairs, then perform `arp-scan`, and finally rewrite `/etc/hosts.home` then restart `dnsmasq`

So our hosts file `/etc/hosts.home` should be like everyone expects it:

```bash
192.168.0.109   wksdev0002
192.168.0.110   wksdev0003
```

## DNS names reesolution tests

If everything was correcty configured, your output should be like this:

```bash
[maksym@srvgen0001 Scripts]$ nslookup wksdev0001.corp.maks-it.com
Server:		127.0.0.1
Address:	127.0.0.1#53

Name:	wksdev0001.corp.maks-it.com
Address: 192.168.0.2
```

## Afterwords

I'm not good in bash scripting and when I see how to loop arrays whith it, my heart starts bleading... So here is some .Net example, maybe someone of you decide to write it in even more Linux Admin way...:

**appsettings.json**

```json
{
    "ServiceSettings": {
        "Dnsmasq": {
            "Bindings": {
                "wksdev0002": "b8:8d:00:00:00:00"
            },
            "AddnHostsPath": "/etc/hosts.home",
            "Interval": 15
        }
    }
}
```

**ServiceSettings.cs**

```csharp
using System.Collections.Generic;

namespace LinuxAdmin.Helpers {
    public class ServiceSettings {
        public Dnsmasq Dnsmasq { get; set; }
    }

    public class Dnsmasq {
        public Dictionary<string, string> Bindings { get; set; }
        public string AddnHostsPath { get; set; }
        public int Interval { get; set; }
    }
}
```

**Program.cs**

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

using LinuxAdmin.Services;
using LinuxAdmin.Helpers;

namespace LinuxAdmin
{
    class Program
    {
        static async Task Main(string[] args)
        { 
            //setup our DI
            var configuration = Configuration();
            var services = new ServiceCollection();

            // configure strongly typed settings objects
            var appSettingsSection = configuration.GetSection("ServiceSettings");
            services.Configure<ServiceSettings>(appSettingsSection);
            var appSettings = appSettingsSection.Get<ServiceSettings>();

            services.AddScoped<IShellService, ShellService>();
            services.AddScoped<INetorkService, NetorkService>();
            services.AddScoped<IDnsmasqService, DnsmasqService>();

            using(ServiceProvider serviceProvider = services.BuildServiceProvider()){
                await DnsmasqUpdate(TimeSpan.FromSeconds(appSettings.Interval), serviceProvider);
            }
        }

        private static IConfigurationRoot Configuration()
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(AppDomain.CurrentDomain.BaseDirectory)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true);
            return builder.Build();
        }

        public static async Task DnsmasqUpdate(TimeSpan interval, IServiceProvider serviceProvider, CancellationToken cancellationToken = new CancellationToken())
        {
            IDnsmasqService _dnsmasqService = serviceProvider.GetService<IDnsmasqService>();

            while (true)
            {
                await Task.Run(() =>
                {
                    Console.WriteLine("The Elapsed event A was raised at {0}", DateTime.Now);
                    _dnsmasqService.AddnHostsUpdate();
                });
                await Task.Delay(interval, cancellationToken);
            }
        }
    }
}

```

**ShellService.cs**

```csharp
using System.Diagnostics;

namespace LinuxAdmin.Services {

    public interface IShellService {
        string RunProcess (string fileName, string args);
    }

    public class ShellService : IShellService {
       public string RunProcess (string fileName, string args) {

            var p = new Process()
            {
                StartInfo = new ProcessStartInfo
                {
                    FileName = fileName,
                    Arguments = args,
                    RedirectStandardOutput = true,
                    UseShellExecute = false,
                    CreateNoWindow = true,
                }
            };
            p.Start();

            // To avoid deadlocks, always read the output stream first and then wait.  
            string output = p.StandardOutput.ReadToEnd();
            p.WaitForExit();

            return output;
        } 
    }
}
```

**NetworkService.cs**

```csharp
using System;
using System.Text.RegularExpressions;
using System.Collections.Generic;

namespace LinuxAdmin.Services {
    
    public interface INetorkService {
        Dictionary<string, string> ArpScan ();
    }


    public class NetorkService : INetorkService {
        
        private readonly IShellService _shellService;
        
        public NetorkService(IShellService shellService) {
            _shellService = shellService;
        }
        public Dictionary<string, string> ArpScan () {
            Console.WriteLine("arp-scan -l result:");
            var result = new Dictionary<string, string>();

            var regex = @"\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b.*\b([0-9A-Fa-f]{2}[:-]){5}([0-9A-Fa-f]{2})\b";

            var output = _shellService.RunProcess("arp-scan", "-l");
            foreach(var row in output.Split('\n')) {
                var rx = new Regex(regex);
                if(rx.Matches(row).Count > 0) {
                    Console.WriteLine(row);

                    var splitRow = row.Split('\t');
                    result.Add(splitRow[0].Trim(), splitRow[1].Trim());
                }
            }

            return result;
        }
    }
}
```

**DnsmasqService.cs**

```csharp
using System;
using System.IO;
using System.Linq;
using System.Collections.Generic;
using Microsoft.Extensions.Options;
using LinuxAdmin.Helpers;

namespace LinuxAdmin.Services {
    
    public interface IDnsmasqService {
        void AddnHostsUpdate ();
    }

    public class DnsmasqService : IDnsmasqService {
        private readonly Dnsmasq _serviceSettings;
        private readonly INetorkService _networkService;
        private readonly IShellService _shellService;

        public DnsmasqService(IOptions<ServiceSettings> serviceSettings, INetorkService networkService, IShellService shellService) {
            _serviceSettings = serviceSettings.Value.Dnsmasq;
            _networkService = networkService;
            _shellService = shellService;
        }

        public void AddnHostsUpdate () {

            var AddnHostsBindings = new Dictionary<string, string>();
            foreach(var machine in _networkService.ArpScan()) {
                var machineName = _serviceSettings.Bindings.Where(x => x.Value == machine.Value).Select(x => x.Key).FirstOrDefault();

                if(machineName != null)
                {
                    AddnHostsBindings.Add(machine.Key, machineName);
                }
            }

            Console.WriteLine($"addn-hosts={_serviceSettings.AddnHostsPath} content:");
            using (StreamWriter file = new StreamWriter(_serviceSettings.AddnHostsPath))
            foreach (var entry in AddnHostsBindings) {
                var line = $"{entry.Key}\t{entry.Value}";
                file.WriteLine(line);
                Console.WriteLine(line);
            }

            Console.WriteLine(_shellService.RunProcess("systemctl", "restart dnsmasq"));
            Console.WriteLine(_shellService.RunProcess("systemctl", "status dnsmasq"));
        }
    }
}
```
## Afterwards v0.1

Solution works, but it is always very ugly, as you need to perform `dnsmasq` restart each time, should be better to include `arp-scan` into `dnsmasq` source code and perform network scans inside its own process without interruptions. Yeah! Instructive story...