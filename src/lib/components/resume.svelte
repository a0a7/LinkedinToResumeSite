<script lang="ts">
    import ResumeHeader from "$lib/components/resume/resume-header.svelte";
    import ResumeSection from "$lib/components/resume/resume-section.svelte";
    import { ExternalLinkIcon } from "lucide-svelte";

    type Contact = {
        icon: string;
        label: string;
        href?: string;
    };

    type Props = {
        data: any;
        contacts: Contact[];
    };

    let { data, contacts }: Props = $props();

    function formatDate(dateStr: string) {
        if (!dateStr) return '';
        const date = new Date(dateStr);
        return date.toLocaleDateString('en-US', { month: 'short', year: 'numeric' });
    }

    function formatDateRange(startDate: string, endDate: string) {
        const start = formatDate(startDate);
        const end = endDate && endDate !== '' ? formatDate(endDate) : 'Present';
        return `${start} - ${end}`;
    }
</script>

<div class="mx-auto max-w-3xl space-y-6 p-8">
    {#if data.basics}
        <ResumeHeader
            name={data.basics.name || "Name"}
            title={data.basics.label || "Professional Title"}
            {contacts}
        />
    {/if}

    {#if data.basics?.summary}
        <ResumeSection title="Summary">
            <p class="text-sm">{data.basics.summary}</p>
        </ResumeSection>
    {/if}

    {#if data.work && data.work.length > 0}
        <ResumeSection title="Experience">
            <div class="space-y-4">
                {#each data.work as job}
                    <div class="group relative">
                        <div class="flex justify-between">
                            <div>
                                <h3 class="font-medium">{job.position || "Position"}</h3>
                                <p class="text-sm text-muted-foreground">{job.name || job.company || "Company"}</p>
                            </div>
                            <div class="text-right text-sm text-muted-foreground">
                                {#if job.location}
                                    <div>{job.location}</div>
                                {/if}
                                {#if job.startDate}
                                    <div>{formatDateRange(job.startDate, job.endDate)}</div>
                                {/if}
                            </div>
                        </div>
                        {#if job.summary}
                            <p class="mt-2 text-sm">{job.summary}</p>
                        {/if}
                        {#if job.highlights && job.highlights.length > 0}
                            <ul class="mt-2 list-inside list-disc text-sm">
                                {#each job.highlights as highlight}
                                    <li>{highlight}</li>
                                {/each}
                            </ul>
                        {/if}
                    </div>
                {/each}
            </div>
        </ResumeSection>
    {/if}

    {#if data.volunteer && data.volunteer.length > 0}
        <ResumeSection title="Volunteer Experience">
            <div class="space-y-4">
                {#each data.volunteer as vol}
                    <div class="group relative">
                        <div class="flex justify-between">
                            <div>
                                <h3 class="font-medium">{vol.position || "Position"}</h3>
                                <p class="text-sm text-muted-foreground">{vol.organization || "Organization"}</p>
                            </div>
                            <div class="text-right text-sm text-muted-foreground">
                                {#if vol.location}
                                    <div>{vol.location}</div>
                                {/if}
                                {#if vol.startDate}
                                    <div>{formatDateRange(vol.startDate, vol.endDate)}</div>
                                {/if}
                            </div>
                        </div>
                        {#if vol.summary}
                            <p class="mt-2 text-sm">{vol.summary}</p>
                        {/if}
                        {#if vol.highlights && vol.highlights.length > 0}
                            <ul class="mt-2 list-inside list-disc text-sm">
                                {#each vol.highlights as highlight}
                                    <li>{highlight}</li>
                                {/each}
                            </ul>
                        {/if}
                    </div>
                {/each}
            </div>
        </ResumeSection>
    {/if}

    {#if data.education && data.education.length > 0}
        <ResumeSection title="Education">
            <div class="space-y-4">
                {#each data.education as edu}
                    <div class="group relative">
                        <div class="flex justify-between">
                            <div>
                                <h3 class="font-medium">{edu.institution || "Institution"}</h3>
                                {#if edu.studyType || edu.area}
                                    <p class="text-sm">{edu.studyType || ""}{edu.area ? (edu.studyType ? ` in ${edu.area}` : edu.area) : ""}</p>
                                {/if}
                            </div>
                            <div class="text-right text-sm text-muted-foreground">
                                {#if edu.startDate}
                                    <div>{formatDateRange(edu.startDate, edu.endDate)}</div>
                                {/if}
                            </div>
                        </div>
                        {#if edu.summary}
                            <p class="mt-2 text-sm">{edu.summary}</p>
                        {/if}
                        {#if edu.courses && edu.courses.length > 0}
                            <ul class="mt-2 list-inside list-disc text-sm">
                                {#each edu.courses as course}
                                    <li>{course}</li>
                                {/each}
                            </ul>
                        {/if}
                    </div>
                {/each}
            </div>
        </ResumeSection>
    {/if}

    {#if data.skills && data.skills.length > 0}
        <ResumeSection title="Skills">
            <div class="space-y-2">
                {#each data.skills as skill}
                    <div class="flex gap-2">
                        <span class="font-medium">{skill.name}:</span>
                        <span class="text-muted-foreground">
                            {#if skill.keywords && skill.keywords.length > 0}
                                {skill.keywords.join(', ')}
                            {:else}
                                {skill.level || ''}
                            {/if}
                        </span>
                    </div>
                {/each}
            </div>
        </ResumeSection>
    {/if}

    {#if data.awards && data.awards.length > 0}
        <ResumeSection title="Awards & Honors">
            <div class="space-y-3">
                {#each data.awards as award}
                    <div>
                        <div class="flex justify-between">
                            <h3 class="font-medium">{award.title || "Award"}</h3>
                            {#if award.date}
                                <span class="text-sm text-muted-foreground">{formatDate(award.date)}</span>
                            {/if}
                        </div>
                        {#if award.awarder}
                            <p class="text-sm text-muted-foreground">{award.awarder}</p>
                        {/if}
                        {#if award.summary}
                            <p class="mt-1 text-sm">{award.summary}</p>
                        {/if}
                    </div>
                {/each}
            </div>
        </ResumeSection>
    {/if}

    {#if data.certificates && data.certificates.length > 0}
        <ResumeSection title="Certifications">
            <div class="space-y-3">
                {#each data.certificates as cert}
                    <div>
                        <div class="flex justify-between">
                            <h3 class="font-medium">{cert.name || "Certificate"}</h3>
                            {#if cert.date}
                                <span class="text-sm text-muted-foreground">{formatDate(cert.date)}</span>
                            {/if}
                        </div>
                        {#if cert.issuer}
                            <p class="text-sm text-muted-foreground">{cert.issuer}</p>
                        {/if}
                        {#if cert.url}
                            <a href={cert.url} class="text-sm text-blue-600 hover:underline flex items-center gap-1">
                                View Certificate <ExternalLinkIcon class="h-3 w-3" />
                            </a>
                        {/if}
                    </div>
                {/each}
            </div>
        </ResumeSection>
    {/if}

    {#if data.publications && data.publications.length > 0}
        <ResumeSection title="Publications">
            <div class="space-y-3">
                {#each data.publications as pub}
                    <div>
                        <div class="flex justify-between">
                            <h3 class="font-medium">{pub.name || "Publication"}</h3>
                            {#if pub.releaseDate}
                                <span class="text-sm text-muted-foreground">{formatDate(pub.releaseDate)}</span>
                            {/if}
                        </div>
                        {#if pub.publisher}
                            <p class="text-sm text-muted-foreground">{pub.publisher}</p>
                        {/if}
                        {#if pub.summary}
                            <p class="mt-1 text-sm">{pub.summary}</p>
                        {/if}
                        {#if pub.url}
                            <a href={pub.url} class="text-sm text-blue-600 hover:underline flex items-center gap-1">
                                View Publication <ExternalLinkIcon class="h-3 w-3" />
                            </a>
                        {/if}
                    </div>
                {/each}
            </div>
        </ResumeSection>
    {/if}

    {#if data.projects && data.projects.length > 0}
        <ResumeSection title="Projects">
            <div class="space-y-4">
                {#each data.projects as project}
                    <div>
                        <div class="flex justify-between">
                            <h3 class="font-medium">{project.name || "Project"}</h3>
                            {#if project.startDate}
                                <span class="text-sm text-muted-foreground">{formatDateRange(project.startDate, project.endDate)}</span>
                            {/if}
                        </div>
                        {#if project.description}
                            <p class="mt-1 text-sm">{project.description}</p>
                        {/if}
                        {#if project.highlights && project.highlights.length > 0}
                            <ul class="mt-2 list-inside list-disc text-sm">
                                {#each project.highlights as highlight}
                                    <li>{highlight}</li>
                                {/each}
                            </ul>
                        {/if}
                        {#if project.url}
                            <a href={project.url} class="text-sm text-blue-600 hover:underline flex items-center gap-1">
                                View Project <ExternalLinkIcon class="h-3 w-3" />
                            </a>
                        {/if}
                    </div>
                {/each}
            </div>
        </ResumeSection>
    {/if}

    {#if data.languages && data.languages.length > 0}
        <ResumeSection title="Languages">
            <div class="space-y-1">
                {#each data.languages as lang}
                    <div class="flex gap-2">
                        <span class="font-medium">{lang.language}:</span>
                        <span class="text-muted-foreground">{lang.fluency}</span>
                    </div>
                {/each}
            </div>
        </ResumeSection>
    {/if}

    {#if data.interests && data.interests.length > 0}
        <ResumeSection title="Interests">
            <div class="space-y-1">
                {#each data.interests as interest}
                    <div>
                        <span class="font-medium">{interest.name}</span>
                        {#if interest.keywords && interest.keywords.length > 0}
                            <span class="text-muted-foreground">: {interest.keywords.join(', ')}</span>
                        {/if}
                    </div>
                {/each}
            </div>
        </ResumeSection>
    {/if}
</div>