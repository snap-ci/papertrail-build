require 'rubygems'
require 'bundler/setup'

package_name = 'remote_syslog2'
version      = '0.13'

release = Time.now.utc.strftime('%Y%m%d%H%M%S')

dirs = ['log', 'pkg', 'jailed-root', 'downloads']

task :clean do
  rm_rf dirs
end

task :init do
  mkdir_p dirs
end

task :download do
  cd 'downloads' do
    sh("curl --fail --silent --location --remote-name https://github.com/papertrail/remote_syslog2/releases/download/v#{version}/remote_syslog_linux_amd64.tar.gz")
    sh('tar -zxf remote_syslog_linux_amd64.tar.gz')
  end
end

task :build do
  require 'erb'

  mkdir_p 'jailed-root/usr/sbin'
  mkdir_p 'jailed-root/etc/init.d'
  mkdir_p 'jailed-root/etc/papertrail'

  cp 'downloads/remote_syslog/remote_syslog', 'jailed-root/usr/sbin'

  ERB.new(File.read('init-script.erb'), nil , '-').tap do |template|
    File.open('jailed-root/etc/init.d/papertrail', 'w') do |f|
      f.puts(template.result)
    end
  end

  chmod 0755, "jailed-root/etc/init.d/papertrail"
end

task :package, [:dist] do |t, args|

  dist = args[:dist] || 'deb'
  description_string = %Q{Lightweight daemon to tail one or more log files and transmit UDP syslog messages to a remote syslog host (centralized log aggregation). Generates UDP packets itself instead of depending on a system syslog daemon, so it doesn't affect system-wide logging configuration.}

  File.open("jailed-root/etc/papertrail/logs.yml", 'w') do |f|
    f.puts %Q{# File generated by #{package_name} #{dist}, modify it there.}
    f.puts %Q{# For examples see https://github.com/papertrail/remote_syslog/tree/master/examples}
    f.puts %Q{files:}
    f.puts %Q{  - /dev/null}
    f.puts %Q{destination:}
    f.puts %Q{  host: example.papertrailapp.com}
    f.puts %Q{  port: 12345}
  end
  sh(%Q{bundle exec fpm -s dir -t #{dist} --force --package pkg/#{package_name}-#{version}-#{release}.x86_64.#{dist} --name #{package_name} -a x86_64 --version "#{version}" -C jailed-root --verbose --#{dist}-user root --#{dist}-group root --config-files etc/papertrail/logs.yml --maintainer support@snap-ci.com --vendor https://github.com/papertrail/remote_syslog2 --url https://github.com/papertrail/remote_syslog2 --description #{Shellwords.escape(description_string)} --iteration #{release} --license 'https://github.com/papertrail/remote_syslogs/blob/v#{version}/LICENSE' .})
end

desc "build rpm and package #{package_name}-#{version}"
task :all, [:dist] => [:clean, :init, :download, :build, :package]

desc "build all #{package_name}"
task :default => :all
