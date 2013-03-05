newrelic-cap-multistage
=======================

Quick recipe to use newrelic with capistrano in a multistaged setup

=======================

# 
# = Capistrano newrelic.rb task
#
# Provides a couple of tasks for creating the newreclic.yml
# in a multistaged environment 
# 

namespace :deploy do

  namespace :newrelic do

    desc <<-DESC
      Creates the newrelic.yml configuration file in shared path
      by copying the newrelic.yml.sample file in /config

      When this recipe is loaded, newrelic:setup is automatically configured \
      to be invoked after deploy:setup. 
    DESC
    task :setup, :except => { :no_release => true } do
      destination_file = "#{shared_path}/config/newrelic.yml"
      
      if remote_file_exists?(destination_file)
        puts "NewRelic file exists for: "+destination_file+"...so skipping..."
      else
        
        location = fetch(:template_dir, "config") + '/newrelic.yml.sample'
        f = File.read(location)

        config = YAML::load(f)
        config["production"]["app_name"] += "::#{stage}"

        run "mkdir -p #{shared_path}/config" 
        put YAML::dump(config), "#{shared_path}/config/newrelic.yml"
      end
    end

    desc <<-DESC
      [internal] Updates the symlink for credentials.yml file to the just deployed release.
    DESC
    task :symlink, :except => { :no_release => true } do
      run "ln -nfs #{shared_path}/config/newrelic.yml #{release_path}/config/newrelic.yml" 
    end

  end

  after "deploy:setup",           "deploy:newrelic:setup"
  after "deploy:finalize_update", "deploy:newrelic:symlink"

end
