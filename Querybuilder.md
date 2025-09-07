package com.example.sqlgenerator;

import jakarta.persistence.Entity;
import jakarta.persistence.Table;

import java.lang.reflect.Field;
import java.util.*;
import java.util.stream.Collectors;

/**
 * SQL Query Builder for JPA entities with JOIN support
 */
public class EntityQueryBuilder<T> {

    private final Class<T> entityClass;
    private final String tableName;
    private final String tableAlias;
    private final List<String> selectFields;
    private final List<String> whereConditions = new ArrayList<>();
    private final List<String> joins = new ArrayList<>();
    private String orderBy;
    private Integer limit;
    private Integer offset;

    public EntityQueryBuilder(Class<T> entityClass, String alias) {
        if (!entityClass.isAnnotationPresent(Entity.class)) {
            throw new IllegalArgumentException("Class " + entityClass.getSimpleName() + " is not a JPA Entity.");
        }
        this.entityClass = entityClass;

        // Table name
        if (entityClass.isAnnotationPresent(Table.class)) {
            Table table = entityClass.getAnnotation(Table.class);
            this.tableName = !table.name().isEmpty() ? table.name() : entityClass.getSimpleName();
        } else {
            this.tableName = entityClass.getSimpleName();
        }

        this.tableAlias = alias;

        // Fields with alias prefix
        this.selectFields = Arrays.stream(entityClass.getDeclaredFields())
                .map(Field::getName)
                .map(field -> alias + "." + field + " AS " + alias + "_" + field)
                .collect(Collectors.toList());
    }

    /** Add WHERE condition */
    public EntityQueryBuilder<T> where(String condition) {
        this.whereConditions.add(condition);
        return this;
    }

    /** Add ORDER BY clause */
    public EntityQueryBuilder<T> orderBy(String column, boolean ascending) {
        this.orderBy = column + (ascending ? " ASC" : " DESC");
        return this;
    }

    /** Add LIMIT clause */
    public EntityQueryBuilder<T> limit(int limit) {
        this.limit = limit;
        return this;
    }

    /** Add OFFSET clause */
    public EntityQueryBuilder<T> offset(int offset) {
        this.offset = offset;
        return this;
    }

    /** Add JOIN clause */
    public <J> EntityQueryBuilder<T> join(Class<J> joinEntity, String alias, String joinCondition, String joinType) {
        String joinTable;
        if (joinEntity.isAnnotationPresent(Table.class)) {
            Table table = joinEntity.getAnnotation(Table.class);
            joinTable = !table.name().isEmpty() ? table.name() : joinEntity.getSimpleName();
        } else {
            joinTable = joinEntity.getSimpleName();
        }

        joins.add(joinType + " JOIN " + joinTable + " " + alias + " ON " + joinCondition);

        // Also add join fields to SELECT
        Arrays.stream(joinEntity.getDeclaredFields())
                .map(Field::getName)
                .map(field -> alias + "." + field + " AS " + alias + "_" + field)
                .forEach(selectFields::add);

        return this;
    }

    /** Shortcut for INNER JOIN */
    public <J> EntityQueryBuilder<T> innerJoin(Class<J> joinEntity, String alias, String condition) {
        return join(joinEntity, alias, condition, "INNER");
    }

    /** Shortcut for LEFT JOIN */
    public <J> EntityQueryBuilder<T> leftJoin(Class<J> joinEntity, String alias, String condition) {
        return join(joinEntity, alias, condition, "LEFT");
    }

    /** Build final SQL query */
    public String build() {
        StringBuilder query = new StringBuilder();
        query.append("SELECT ")
                .append(String.join(", ", selectFields))
                .append(" FROM ").append(tableName).append(" ").append(tableAlias);

        if (!joins.isEmpty()) {
            query.append(" ").append(String.join(" ", joins));
        }

        if (!whereConditions.isEmpty()) {
            query.append(" WHERE ").append(String.join(" AND ", whereConditions));
        }

        if (orderBy != null) {
            query.append(" ORDER BY ").append(orderBy);
        }

        if (limit != null) {
            query.append(" LIMIT ").append(limit);
        }

        if (offset != null) {
            query.append(" OFFSET ").append(offset);
        }

        query.append(";");
        return query.toString();
    }
}
