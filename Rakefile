DOC_FILES = FileList.new(File.join('docs', '**', '*.md'))
SPEC_FILES = FileList.new(File.join('reference', '*.yaml'))

TEMP_DIR = 'temp'
DIST_DIR = 'dist'
POSTMAN_DIR = File.join(DIST_DIR, 'test', 'postman')

def bundled(spec)
    File.join(TEMP_DIR, spec.to_s.pathmap('%f'))
end

def folder(spec, package, format)
    File.join(DIST_DIR, package.to_s, format.to_s, spec.to_s.pathmap('%n'))
end

task :default => :all

task :all =>  [ :validate, :generate ]

desc "clean intermediate files"
task :clean do
	rm_rf TEMP_DIR
end

desc "clobber all generated files"
task :clobber => [ :clean ] do
    rm_rf DIST_DIR
end

namespace :bundle do
    directory TEMP_DIR

    SPEC_FILES.each do |spec|
        file bundled(spec) => [ TEMP_DIR, spec ] do
            sh "swagger-cli bundle --dereference --type yaml --outfile #{bundled(spec)} #{spec}"
        end

        desc "bundle specs"
        task :specs => [ bundled(spec) ]
    end
end

task :validate => 'validate:all'

namespace :validate do
    namespace :docs do
        DOC_FILES.each do |doc|
            desc "validate docs with markdownlint"
            task :markdownlint => [ doc ] do
                sh "markdownlint #{doc}"
            end

        end

        desc "validate docs with all tools"
        task :all => [ 'markdownlint' ]
    end

    namespace :specs do
        SPEC_FILES.each do |spec|
            desc "validate specs with Stoplight Spectral"
            task :spectral => [ spec ] do
                sh "spectral lint --verbose #{spec}"
            end

            desc "validate specs with Swagger CLI"
            task :swagger_cli => [ spec ] do
                sh "swagger-cli validate #{spec}"
            end

            desc "validate specs with OpenAPI Generator"
            task :openapi_generator => [ bundled(spec) ] do
                sh "openapi-generator-cli validate --input-spec #{bundled(spec)}"
            end
        end

        desc "validate specs with all tools"
        task :all => [ :spectral, :swagger_cli, :openapi_generator ]
    end

    desc "validate all files"
    task :all => [ 'docs:all', 'specs:all' ]
end

task :generate => [ 'generate:all' ]

namespace :generate do
    directory DIST_DIR

    PACKAGES = {
        docs: {
            html: 'html2',
        },
        client: {
            javascript: 'javascript',
            typescript: 'typescript-axios',
            # python: 'python-legacy', # both 'python' and 'python-legacy' barfs on the clinic spec
            swift: 'swift5',
            ruby: 'ruby',
            lua: 'lua',
            go: 'go',
        },
        server: {
            go: 'go-server',
        }
    }
    PACKAGES.each do |package, formats|
        namespace package do
            formats.each do |format, generator|
                SPEC_FILES.each do |spec|
                    output = folder(spec, package, format)
                    directory output

                    desc "generate #{package} in #{format} using generator '#{generator}'"
                    task format => [ bundled(spec), output ] do
                        sh "openapi-generator-cli generate --input-spec #{bundled(spec)} --generator-name #{generator} --output #{output} > #{File.join(output, 'generator.log')}"
                    end

                    desc "generate #{package} for all formats: #{formats.keys.map { |l| l.to_s }.join(', ')}"
                    task :all => [ format ]
                end
            end
        end

        desc "generate all of #{PACKAGES.keys.map { |p| p.to_s }.join(', ')}"
        task :all => [ "#{package}:all" ]
    end

    SPEC_FILES.each do |spec|
        directory POSTMAN_DIR

        output = File.join(POSTMAN_DIR, spec.to_s.pathmap('%n.json'))
        file output => [ bundled(spec), POSTMAN_DIR ] do
            sh "openapi2postmanv2 --spec #{bundled(spec)} --output #{output} --pretty"
        end

        desc "generate Postman Collection"
        task :postman => output
    end

    task :all => [ :postman ]
end
