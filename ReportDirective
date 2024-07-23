<?php

declare(strict_types=1);

namespace Webard\LighthouseBiloquent\GraphQL\Directives;

use GraphQL\Type\Definition\ResolveInfo;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Support\Collection;
use LastDragon_ru\LaraASP\GraphQL\Builder\BuilderInfo;
use LastDragon_ru\LaraASP\GraphQL\Builder\Contracts\BuilderInfoProvider;
use LastDragon_ru\LaraASP\GraphQL\Builder\Contracts\TypeSource;
use Nuwave\Lighthouse\Schema\Directives\BaseDirective;
use Nuwave\Lighthouse\Schema\Values\FieldValue;
use Nuwave\Lighthouse\Support\Contracts\FieldResolver;
use Nuwave\Lighthouse\Support\Contracts\GraphQLContext;
use Webard\Biloquent\Report;

class ReportDirective extends BaseDirective implements BuilderInfoProvider, FieldResolver
{
    /**
     * @var array<mixed>
     */
    private array $filters;

    /**
     * @var array<int,string>
     */
    private array $groups;

    public static function definition(): string
    {
        return /** @lang GraphQL */ <<<'CODE_SAMPLE'
            """
            Run report class.
            """
            directive @report(
              """
              Specify the class name of the model to use.
              This is only needed when the default model detection does not work.
              """
              model: String
            ) on FIELD_DEFINITION
            CODE_SAMPLE;
    }

    public function getBuilderInfo(TypeSource $source): ?BuilderInfo
    {
        return new BuilderInfo('raport', Builder::class);
    }

    public function resolveField(FieldValue $fieldValue): callable
    {
        return function (
            $root,
            array $args,
            GraphQLContext $context,
            ResolveInfo $resolveInfo
        ): Collection {
            $this->filters = $args['filter'] ?? [];
            $this->groups = $args['group'] ?? [];

            $rootFieldSelection = $resolveInfo->getFieldSelection(0);

            /** @var array<string> $rootFields */
            $rootFields = array_keys($rootFieldSelection);

            /** @var Report $class */
            $class = $this->getModelClass();

            $this->groups = (new Collection($this->groups))->transform(fn ($value) => mb_strtolower($value))->toArray();

            //@phpstan-ignore-next-line
            $q = (new $class())->query()->enhance(
                fn (Builder $query) =>
            //@phpstan-ignore-next-line
            $query->filter($this->filters)
            )->grouping($this->groups)->columns($rootFields)->prepare();

            return $q->get();
        };
    }
}
