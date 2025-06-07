<script lang="ts">
        import { Card, CardContent } from "$lib/components/ui/card";
        import { Separator } from "$lib/components/ui/separator";
        import { PhoneIcon, GlobeIcon, MailIcon, MapPinIcon, BriefcaseIcon, GraduationCapIcon, UserIcon, MessagesSquareIcon, CircleIcon, GithubIcon, Github } from "lucide-svelte";
        
        // Resume data
        const personalInfo = {
            name: "Alexander Weimer",
            title: "Computer Engineering Student @ University of Minnesota",
            phone: "+1 (612) 743-0320",
            email: "aw@a0.ax",
            address: "", // Empty to test hiding
            website: "https://a0.ax",
            github: "github.com/a0a7"
        };

        const skills = [
            "Strategic Planning",
            "Problem Solving",
            "Crisis Management", 
            "Creative Thinking",
            "Data Analysis",
            "Brand Development",
            "Negotiation",
            "Customer Orientation",
            "Adaptability to Change"
        ];

        const languages = [
            "English (Native)",
            "Swedish (Native)",
            "German (Advanced)"
        ];

        const awards = [
            {
                title: "Dean's List",
                issuer: "University of Minnesota",
                date: "2023",
                description: "Achieved academic excellence with GPA above 3.75"
            },
            {
                title: "Outstanding Student Leader Award",
                issuer: "Engineering Student Society",
                date: "2024",
                description: "Recognized for exceptional leadership in student organizations"
            },
            {
                title: "Innovation Challenge Winner",
                issuer: "Tech Innovation Lab",
                date: "2024",
                description: "First place in university-wide technology innovation competition"
            }
        ];

        const workExperience = [
            {
                company: "Borcelle Studio",
                role: "Marketing Manager & Specialist",
                period: "2030 - PRESENT",
                duties: [
                    "Formulate and implement detailed marketing strategies and initiatives that support the company's mission and objectives.",
                    "Guide, inspire, and oversee a dynamic marketing team, promoting a collaborative and performance-oriented culture.",
                    "Ensure uniformity of the brand across all marketing platforms and materials."
                ]
            },
            {
                company: "Pauget Studio",
                role: "Marketing Manager & Specialist", 
                period: "2025 - 2029",
                duties: [
                    "Neimo ensiont est atem Aham ipsum, finifam imbo sido. Intum lumina parco regia Aham extra mola idis. Commodo Aham antepor yumo finiam carius, domus serim lacur Aham. Mauris solitudo Aham in albaidis, ceras vita Aham elit exidio leo, adipisci sector elit."
                ]
            },
            {
                company: "Studio Shodwe",
                role: "Marketing Manager & Specialist",
                period: "2024 - 2025",
                duties: [
                    "Formulate and implement detailed marketing strategies and initiatives that support the company's mission and objectives. Guide, inspire, and oversee a dynamic marketing team, promoting a collaborative and performance-oriented culture. Ensure uniformity of the brand across all marketing platforms and materials."
                ]
            }
        ];

        const education = {
            degree: "Bachelor of Electrical Engineering",
            school: "University of Minnesota",
            period: "2025-2028 (estimated)"
        };

        const reference = {
        /*    name: "Estelle Darcy",
            title: "Wardiere Inc. / CTO",
            phone: "+124-4236-7894",
            email: "hello@ahmedd saaahh.com"   */
        };
    
    </script>

    <div class="min-h-screen bg-background p-8 print:p-4 print:min-h-0">
        <Card class="mx-auto max-w-4xl">
            <CardContent class="p-8 print:p-4">
                <!-- Header -->
                <div class="mb-8 text-center items-center w-full">
                    {#if personalInfo.name}
                        <h1 class="text-4xl font-bold print:text-3xl">{personalInfo.name}</h1>
                    {/if}
                    {#if personalInfo.title}
                        <h2 class="text-xl text-muted-foreground print:text-lg">{personalInfo.title}</h2>
                    {/if}
                    
                    {#if personalInfo.phone || personalInfo.email || personalInfo.address || personalInfo.website}
                        <div class="flex flex-wrap gap-4 mt-4 w-full justify-center items-center print:gap-2">
                            {#if personalInfo.phone}
                                <div class="flex items-center gap-1">
                                    <PhoneIcon class="h-4 w-4" />
                                    <span class="text-sm">{personalInfo.phone}</span>
                                </div>
                            {/if}
                            {#if personalInfo.email}
                                <div class="flex items-center gap-1">
                                    <MailIcon class="h-4 w-4" />
                                    <span class="text-sm">{personalInfo.email}</span>
                                </div>
                            {/if}
                            {#if personalInfo.address}
                                <div class="flex items-center gap-1">
                                    <MapPinIcon class="h-4 w-4" />
                                    <span class="text-sm">{personalInfo.address}</span>
                                </div>
                            {/if}
                            {#if personalInfo.website}
                                <div class="flex items-center gap-1">
                                    <GlobeIcon class="h-4 w-4" />
                                    <span class="text-sm">{personalInfo.website}</span>
                                </div>
                            {/if}
                            {#if personalInfo.github}
                                <div class="flex items-center gap-1">
                                    <span class="text-black w-4 h-4">
                                        <svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 256 256"><g><path d="M216,104v8a56.06,56.06,0,0,1-48.44,55.47A39.8,39.8,0,0,1,176,192v40a8,8,0,0,1-8,8H104a8,8,0,0,1-8-8V216H72a40,40,0,0,1-40-40A24,24,0,0,0,8,152a8,8,0,0,1,0-16,40,40,0,0,1,40,40,24,24,0,0,0,24,24H96v-8a39.8,39.8,0,0,1,8.44-24.53A56.06,56.06,0,0,1,56,112v-8a58.14,58.14,0,0,1,7.69-28.32A59.78,59.78,0,0,1,69.07,28,8,8,0,0,1,76,24a59.75,59.75,0,0,1,48,24h24a59.75,59.75,0,0,1,48-24,8,8,0,0,1,6.93,4,59.74,59.74,0,0,1,5.37,47.68A58,58,0,0,1,216,104Z"></path></g></svg>
                                    </span>
                                    <span class="text-sm">{personalInfo.github}</span>
                                </div>
                            {/if}
                        </div>
                    {/if}
                </div>

                <div class="grid grid-cols-1 gap-8 md:grid-cols-3 print:grid-cols-3 print:gap-4">
                    <!-- Left Column -->
                    <div class="space-y-8 print:space-y-4">
                        <!-- Awards moved to left column -->
                        {#if awards && awards.length > 0}
                            <section>
                                <h3 class="mb-4 text-lg font-semibold">AWARDS</h3>
                                <div class="space-y-4">
                                    {#each awards as award}
                                        {#if award.title || award.issuer || award.date || award.description}
                                            <div class="space-y-1">
                                                <div class="flex items-center justify-between">
                                                    {#if award.title}
                                                        <h4 class="font-semibold">{award.title}</h4>
                                                    {/if}
                                                    {#if award.date}
                                                        <span class="text-sm text-muted-foreground">{award.date}</span>
                                                    {/if}
                                                </div>
                                                {#if award.issuer}
                                                    <p class="text-muted-foreground">{award.issuer}</p>
                                                {/if}
                                                {#if award.description}
                                                    <p class="text-sm text-muted-foreground">{award.description}</p>
                                                {/if}
                                            </div>
                                        {/if}
                                    {/each}
                                </div>
                            </section>
                        {/if}

                                                <!-- Languages -->
                        {#if languages && languages.length > 0}
                            <section>
                                <h3 class="mb-4 text-lg font-semibold">LANGUAGES</h3>
                                <ul>
                                    {#each languages as language}
                                        {#if language}
                                            <li>{language}</li>
                                        {/if}
                                    {/each}
                                </ul>
                            </section>
                        {/if}

                        <!-- Skills -->
                        {#if skills && skills.length > 0}
                            <section>
                                <h3 class="mb-4 text-lg font-semibold">Skills</h3>
                                <ul class="space-y-1">
                                    {#each skills as skill}
                                        {#if skill}
                                            <li>{skill}</li>
                                        {/if}
                                    {/each}
                                </ul>
                            </section>
                        {/if}


                        <!-- Reference -->
                        {#if reference && (reference.name || reference.title || reference.phone || reference.email)}
                            <section>
                                <h3 class="mb-4 text-lg font-semibold">REFERENCE</h3>
                                <div class="space-y-2">
                                    {#if reference.name}
                                        <p class="font-semibold">{reference.name}</p>
                                    {/if}
                                    {#if reference.title}
                                        <p>{reference.title}</p>
                                    {/if}
                                    {#if reference.phone}
                                        <div class="flex items-center gap-2">
                                            <PhoneIcon class="h-4 w-4" />
                                            <span>{reference.phone}</span>
                                        </div>
                                    {/if}
                                    {#if reference.email}
                                        <div class="flex items-center gap-2">
                                            <MailIcon class="h-4 w-4" />
                                            <span>{reference.email}</span>
                                        </div>
                                    {/if}
                                </div>
                            </section>
                        {/if}
                    </div>

                    <!-- Right Column -->
                    <div class="space-y-8 md:col-span-2 print:col-span-2 print:space-y-4">
                        <!-- Work Experience -->
                        {#if workExperience && workExperience.length > 0}
                            <section class="flex gap-4">
                                <CircleIcon class="h-4 w-4 shrink-0" />
                                <div class="flex-1">
                                    <h3 class="mb-4 text-lg font-semibold">Experience</h3>
                                    <div class="space-y-6">
                                        {#each workExperience as job}
                                            {#if job.company || job.role || job.period || (job.duties && job.duties.length > 0)}
                                                <div class="space-y-2">
                                                    <div class="flex items-center justify-between">
                                                        {#if job.company}
                                                            <h4 class="font-semibold">{job.company}</h4>
                                                        {/if}
                                                        {#if job.period}
                                                            <span class="text-sm text-muted-foreground">{job.period}</span>
                                                        {/if}
                                                    </div>
                                                    {#if job.role}
                                                        <p class="text-muted-foreground">{job.role}</p>
                                                    {/if}
                                                    {#if job.duties && job.duties.length > 0}
                                                        <ul class="list-inside list-disc space-y-1 text-sm text-muted-foreground">
                                                            {#each job.duties as duty}
                                                                {#if duty}
                                                                    <li>{duty}</li>
                                                                {/if}
                                                            {/each}
                                                        </ul>
                                                    {/if}
                                                </div>
                                            {/if}
                                        {/each}
                                    </div>
                                </div>
                            </section>
                        {/if}

                        <!-- Education -->
                        {#if education && (education.degree || education.school || education.period)}
                            <section class="flex gap-4">
                                <CircleIcon class="h-4 w-4 shrink-0" />
                                <div>
                                    <h3 class="mb-4 text-lg font-semibold">EDUCATION</h3>
                                    <div class="flex items-center justify-between">
                                        <div>
                                            {#if education.degree}
                                                <h4 class="font-semibold">{education.degree}</h4>
                                            {/if}
                                            {#if education.school}
                                                <p class="text-muted-foreground">{education.school}</p>
                                            {/if}
                                        </div>
                                        {#if education.period}
                                            <span class="text-sm text-muted-foreground">{education.period}</span>
                                        {/if}
                                    </div>
                                </div>
                            </section>
                        {/if}
                    </div>
                </div>
            </CardContent>
        </Card>
    </div>

    <style>
        @media print {
            * {
                -webkit-print-color-adjust: exact !important;
                color-adjust: exact !important;
            }
            
            .grid {
                display: grid !important;
            }
            
            .print\:grid-cols-3 {
                grid-template-columns: 1fr 2fr !important;
            }
            
            .print\:col-span-2 {
                grid-column: span 2 / span 2 !important;
            }
            
            .print\:gap-4 {
                gap: 1rem !important;
            }
            
            .print\:space-y-4 > * + * {
                margin-top: 1rem !important;
            }
            
            .print\:p-4 {
                padding: 1rem !important;
            }
            
            .print\:text-3xl {
                font-size: 1.875rem !important;
                line-height: 2.25rem !important;
            }
            
            .print\:text-lg {
                font-size: 1.125rem !important;
                line-height: 1.75rem !important;
            }
            
            .print\:gap-2 {
                gap: 0.5rem !important;
            }
            
            .print\:min-h-0 {
                min-height: 0 !important;
            }
            
            /* Force grid layout */
            @page {
                margin: 0.5in;
            }
            
            /* Ensure proper column layout */
            .md\:col-span-2 {
                grid-column: span 2 / span 2 !important;
            }
        }
    </style>